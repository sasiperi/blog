---
title: Spring Cloud Config Server and Legacy Spring Apps
tags: 
- java
- Spring Boot
- Spring Cloud
- Hybris
categories: developer
desc: Integrating Spring Cloud Config Server with Legacy Spring Apps
layout: post
---

* TOC
{:toc}

1. Step-0: Dependencies
2. Step-1 : Cloud Web Context Class
3. Step-2:  Override Property Locator
4. Step-3:  Config Server Authentication
5. Step-4:  Environment properties
6. Step-5:  All set
7. Hybris specific changes


# Step-0: Dependencies

~~~java
this.threadPoolExecutor = new ThreadPoolExecutor(8, 8, 10L, TimeUnit.SECONDS,
    new LinkedBlockingQueue<>(8),
    new CustomizableThreadFactory("fileSorterTPE-"),
    new ThreadPoolExecutor.CallerRunsPolicy());
~~~

# Step-1 : Cloud Web Context Class

~~~java
List<Future<Chunk>> splitFutureList = new ArrayList<>();
while (true){
    line = br.readLine();
    if(line != null){
        chunkRows.add(line);
    }
    if(line == null || chunkRows.size() >= initialChunkSize){
        if(chunkRows.size() > 0){
            final int rn = rowNum;
            final List<String> cr = chunkRows;
            rowNum += chunkRows.size();
            chunkRows = new ArrayList<>();
            Future<Chunk> chunk = threadPoolExecutor.submit(() -> {
                cr.sort(comparator);
                return initialChunk(rn, cr, file);
            });
            splitFutureList.add(chunk);
        }
    }
    if(line == null){
        break;
    }
}
chunkList = splitFutureList.stream().map(this::get).collect(Collectors.toList());
~~~


# Step-2:  Override Property Locator

~~~java
int currentLevel = INITIAL_CHUNK_LEVEL;
List<Future<Chunk>> mergeFutureList = new ArrayList<>();
while (true) {
    //从队列中获取一组chunk
    List<Chunk> pollChunks = pollChunks(chunkQueue, currentLevel);
    //未取到同级chunk, 表示此级别应合并完成
    if (CollectionUtils.isEmpty(pollChunks)) {
        mergeFutureList.stream().map(this::get).forEach(chunkQueue::add);
        mergeFutureList.clear();
        //chunkQueue 中只有一个元素，表示此次合并是最终合并
        if (chunkQueue.size() == 1) {
            break;
        } else {
            currentLevel++;
            continue;
        }
    }
    Future<Chunk> chunk = threadPoolExecutor.submit(() -> merge(pollChunks, original));
    mergeFutureList.add(chunk);
}
~~~

