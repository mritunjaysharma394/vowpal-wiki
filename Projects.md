1. [VW support for FlatBuff and/or Protobuf](#1-vw-support-for-flatbuff-andor-protobuf)
2. [Parallelized parsing](#2-parallelized-parsing)
3. [VW server mode revamp](#3-vw-server-mode-revamp)
4. [Contextual bandits data visualization with Jupyter notebooks](#4-contextual-bandits-data-visualization-with-jupyter-notebooks)
5. [Improve VW’s Python experience](#5-improve-vws-python-experience)
6. [End-to-load local loop for reinforcement learning](#6-end-to-load-local-loop-for-reinforcement-learning)
7. [TensorWatch and TensorBoard integration](#7-tensorwatch-and-tensorboard-integration)
8. [ONNX operator set and model format for VW models](#8-onnx-operator-set-and-model-format-for-vw-models)
9. [Enable implementation of a VW reduction in Python](#9-enable-implementation-of-a-vw-reduction-in-python)
10. [Allow Python implementations of RLClientLib extensibility points](#10-allow-python-implementations-of-rlclientlib-extensibility-points)
11. [Contextual bandit benchmark and competition](#11-contextual-bandit-benchmark-and-competition)
12. [Library of contextual bandit estimators](#12-library-of-contextual-bandit-estimators)

## 1. VW support for FlatBuff and/or Protobuf
VW has several file inputs, examples, cache and models. This project involves adding support for a modern serialization framework such as FlatBuff or ProtoBuff. This will enable easier interop, better stability and potentially increased performance. 

### Goals
- Produce wiki page outlining design and usage
- Design schemas
- Load and save a model
- Load examples from a file. Start by keeping labels stored as a string.
- Utilities to inspect and convert to/from VW's file formats
- Produce wiki page outlining design and usage

### Stretch Goals
- Load and save the cache file
- Schemas for structured label types
- Benchmark and optimize performance

### Links
- [VW example text input format](https://github.com/VowpalWabbit/vowpal_wabbit/wiki/Input-format)
- [Code pointer for text example parsing](https://github.com/VowpalWabbit/vowpal_wabbit/blob/master/vowpalwabbit/parse_example.cc)
- [Code pointer for model saving and loading](https://github.com/VowpalWabbit/vowpal_wabbit/blob/master/vowpalwabbit/parse_regressor.cc)
- [Flatbuffers GitHub repo](https://github.com/google/flatbuffers)
- [Protobuf GitHub repo](https://github.com/protocolbuffers/protobuf)


## 2. Parallelized parsing
Modern machines often utilize many threads to achieve performance. VW currently uses a single parse thread and a single learner thread, and parsing is often the bottleneck. Extending the parser to support many threads will allow us to better utilize resources.

### Goals
- Produce wiki page outlining design and usage
- Extract parser to standalone component
- Spawn threads for parse jobs
- Ensure original ordering in datafile is preserved 

### Stretch Goals
- Lock free synchronization of threads
- Use all reduce to support multi threaded learning
- Separate I/O threads from parse threads

### Links
- [Code pointer for parser component](https://github.com/VowpalWabbit/vowpal_wabbit/blob/master/vowpalwabbit/parser.cc)
- [Code pointer for text example parsing](https://github.com/VowpalWabbit/vowpal_wabbit/blob/master/vowpalwabbit/parse_example.cc)

## 3. VW server mode revamp
VW currently has daemon mode, which allows clients to send examples, train and model and receive predictions. This uses raw sockets and a custom binary protocol We want to provide a modern version of VW's server mode utilizing a modern RPC technology.

### Goals
- Single model serving using GRPC with the following endpoints:
    - Predict
    - Learn
    - Statistics (number of features, current loss, etc)
    - Management (download current model, number of features)
- Packaging tools to create docker containers from VW params and model
- Wiki page describing how to use it

### Stretch Goals
- Persistent model storage.
- Multiple models from a single daemon.

### Links
- [GRPC](https://grpc.io/)

## 4. Contextual bandits data visualization with Jupyter notebooks 
Build visualizations to help understand the behavior of Contextual Bandit policies and logs. 

### Goals
- Vizualizations for:
    - Action distribution
    - Action/reward distribution by feature(s) or model used
    - Model comparison
    - Feature importance
- Produce a synthetic dataset that highlight the usefulness of the visualizations

### Stretch Goals
- TBD

## 5. Improve VW's Python experience 
VW's Python integration can be improved is several areas to make it easier for users. Supporting Pandas as a first class concept will make utilizing VW in experimentation workflows much more streamlined. Implementing IPython HTML representations for some common types will improve usability of these components.

### Goals
- Implement _repr_html_ for examples, model and labels
- Access to progressive validation and other model statistics
- Pandas load and save from VW text format

### Stretch Goals
- Simplify example lifecycle
### Links
- [IPython HTML representation]https://ipython.readthedocs.io/en/stable/config/integrating.html

## 6. End-to-load local loop for reinforcement learning
The reinforcement learning library has extension points to allow for swapping out parts of the framework, however there is no simple way to make it work end to end locally at the moment. Making RLClientLib support prediction, logging, joining and training locally will make for a great prototyping tool.

### Goals
- In-memory joining and training
- Extend configuration to enable local mode
- Python and C# API support

### Stretch Goals
- Checkpointing - load and save model
- Port some of our RLClientLib simulators to use the local loop
   
### Links
- [RLClientLib GitHub repo](https://github.com/VowpalWabbit/reinforcement_learning)

## 7. TensorWatch and TensorBoard integration
TensorBoard and TensorWatch are great tools for debugging and monitoring training making them a great choice for  integrating with VW and RLClientLib.

### Goals
- Integrate VW training with TensorWatch all within a notebook
- Extend VW to output TensorBoard logs
- Extend RLClientLib to support TensorBoard and TensorWatch
### Stretch Goals
- Add lazy logging mode to VW and RLClientLib

### Links
- [TensorWatch GitHub repo](https://github.com/Microsoft/tensorwatch)
- [TensorBoard](https://www.tensorflow.org/tensorboard/)


## 8. ONNX operator set and model format for VW models
VW has its own runtime for running inference off of its own model files. However, ONNX is the emerging standard for defining models and supporting inference. This project enables VW models to interoperate with ONNX runtime.

### Goals
- Define ONNX.vw operation set for the reductions needed for classification (CSOAA)
- Define shape of VW example in tensor format
- Converter tool from vw model to ONNX model
- Implement the new opset with ONNX runtime
- Sample app that runs inference

### Stretch Goals
- Extend opset to Contextual Bandits
- Export ONNX model directly from VW
### Links
- [ONNX Runtime GitHub repo](https://github.com/Microsoft/onnxruntime)

## 9. Enable implementation of a VW reduction in Python
All reductions in VW are implemented in C++. However, to allow for rapid prototyping and taking advantage of the Python ecosystem, using Python to do this makes sense.

### Goals
- Create interface that allows Python code to implement a base learner in VW
- Implement a simple gradient descent base learner using SKlearn
### Stretch Goals
- Allow for the Python implemented reduction to be used at a different level of the reduction stack
### Links

## 10. Allow Python implementations of RLClientLib extensibility points
RLCLientLib supports several points of extensibility, but these are only exposed in C++. When using RLCLientLib in Python it is important to be able to support these.

### Goals
- Support a custom model implementation in Python through the `i_model` interface
- Create an example of using these locally
### Stretch Goals
- Support custom `i_sender` implementation for event logging 
- Support `i_data_transport` for retrieving updated models

### Links 
- [`i_sender`](https://github.com/VowpalWabbit/reinforcement_learning/blob/master/include/sender.h)
- [`i_data_transport` and `i_model`](https://github.com/VowpalWabbit/reinforcement_learning/blob/master/include/model_mgmt.h)

## 11. Contextual bandit benchmark and competition
There exists many different contextual bandit algorithms. In order to compare these a standard benchmark would be useful. Use the contextual bandit bake off paper as a base and build a set of standard CB benchmarks and supporting infrastructure to competitively evaluate CB algorithms. This is similar to the [GLUE benchmark for NLP](https://gluebenchmark.com/).

### Goals
- Design CB experiments - start off with CB bakeoff paper
- Create infrastructure to obtain datasets
- Upload predictions to evaluate performance of algorithm
- Visualization, display results and compare to others
### Stretch Goals
- Abstract what it means to be a CB algo to provide a more structured evaluation workflow
### Links
- [CB bake off paper](https://arxiv.org/abs/1802.04064)
- [GLUE](https://gluebenchmark.com/)

## 12. Library of contextual bandit estimators
Estimators are used in off policy evaluation. One common estimator is IPS, and others are DR and PseudoInverse. These estimators work better or worse in different settings. This project explores reference implementations of each and allows for comparison between them to aid in understanding. As a stretch goal it involves utilizing this common library of estimators in the existing counterfactual estimation module.

### Goals
- Add implementation of DR, and DR in episodic settings
- Simulator interface that allows evaluation against logging policy and target policy
- Generate a random logging policy and target policy to use for evaluation
- Visualization of comparison 
### Stretch Goals
- Pseudo inverse
- Integrate into existing counterfactual evaluation framework

### Links
- [Estimators GitHub repo](https://github.com/VowpalWabbit/estimators)
