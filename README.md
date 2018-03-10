# SparkOnlineLearning
%md # Description of the ML framework
![My image](bennyavelin.github.com/SparkOnlineLearning/img/StreamingML.svg)

The above diagram describes the process which data is processed using the streaming machine learning framework. We need to be careful with `SStreamingMLAggregator` since this does the aggregation in a way that is natural when you hear it.

![My image](bennyavelin.github.com/SparkOnlineLearning/img/SStreamingMLAggregator.svg)

Whenever a new batch gets processed, it arrives in the input stream and gets split into each `SStreamingMLNode`, each Node does `updateAcrossBatch` and sends an updated output which becomes `Batch1.x`. The first batch gets aggregated into a running state and each new batch arrives, `Batch2` in the above, it gets processed and aggregated ontop of the running state.

This implies that if we wish to train a model in a parallel way, i.e. we have one model per `SStreamingMLNode`, we need to be a bit more clever when we do the final aggregation since we always have a running state when the new batch arrives. I.e. as it is solved in `TestSStreamingMLxx`, we have the running state as a `Map`, keeping track of each `TestSStreamingMLNode`s output and updates that nodes state in the map. 