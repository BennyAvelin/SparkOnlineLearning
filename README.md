# SparkOnlineLearning
## Description of the ML framework
![My image](https://github.com/BennyAvelin/SparkOnlineLearning/blob/master/img/StreamingML.svg)

The above diagram describes the process which data is processed using the streaming machine learning framework. We need to be careful with `SStreamingMLAggregator` since this does the aggregation in a way that is natural when you hear it.

![My image](https://github.com/BennyAvelin/SparkOnlineLearning/blob/master/img/SStreamingMLAggregator.svg)

Whenever a new batch gets processed, it arrives in the input stream and gets split into each `SStreamingMLNode`, each Node does `updateAcrossBatch` and sends an updated output which becomes `Batch1.x`. The first batch gets aggregated into a running state and each new batch arrives, `Batch2` in the above, it gets processed and aggregated ontop of the running state.

This implies that if we wish to train a model in a parallel way, i.e. we have one model per `SStreamingMLNode`, we need to be a bit more clever when we do the final aggregation since we always have a running state when the new batch arrives. I.e. as it is solved in `TestSStreamingMLxx`, we have the running state as a `Map`, keeping track of each `TestSStreamingMLNode`s output and updates that nodes state in the map. 

## Updating the model
* FunctionRegister holds the update functions for use when creating the sink, i.e. we first need to register our specific update function with a unique key value that is later passed as an option when creating the sink.

![My image](https://github.com/BennyAvelin/SparkOnlineLearning/blob/master/img/FunctionRegister.svg)

When we create a `SStreamingML` we need to make sure that in the constructor that the particular update function i.e. `this.update` is registered in the `FunctionRegister`, a template for how this is done can be found in TestSStreamingML. Next whenever we call `fit` on a `SStreamingML` we are creating a `DataStreamWriter[SStreamingML]`, this needs a `Sink` and the way this works is that a `SinkProvider` is called and we can only pass strings as options to this. Thus we have used the functionregister to register the update function that later can be pulled by the SinkProvider based on a unique string sent using an option. Exactly how this works can be seen in the diagram and looking through the example code regarding `TestSStreamingxx`

## Structured streaming machine learning framework
We need the following components:
* A model to store the current state of the model
* A grouping operation, which essentially becomes the partitioning of the data
* An aggregation over the first grouping of the data, this is an aggregation that is run on the node
* An aggregation over the output from the first aggregation.

We can think of this process as splitting the dataset into streams, process each stream individually and return an aggregated output from each stream. Lastly aggregate on the aggregated output to produce a join of the models over all the streams to a single output. 
