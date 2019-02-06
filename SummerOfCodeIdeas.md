This page collects project ideas by VW engineers and researchers. They range from specific engineering/infrastructure all the way to implementing new ML algorithms.

This page should give you a great starting point to submit a project. They are not exhaustive and we'll consider any interesting ideas that aligns with the project goals.

## Create C# VW bindings that work with dotnet core
**Description:** Right now the C# bindings are built with C++/CLI which means they don’t work outside of the desktop framework. Addressing it would enable it to be used in multiple places. It should use C# binding in RL Clientlib as reference to how to do it.

**Deliverables:** A nuget for VW bindings that works under dotnet core.

**Mentors:** Rodrigo / Jacob
 
## Create bindings to different languages
**Description:** Our current VW bindings are only available to Python, Java, C# and C++. There’s a growing demand for ML toolkits from other languages such as Go, Swift or Rust and VW should have good support there.

**Deliverables:** Bindings with samples for one of those languages with the same functionality available under the python binding.

**Mentors:** Rodrigo
 
## Port VW to WebAssembly
**Description:** WebAssembly is the new hot thing with web development and having VW running there would greatly expand our reach.

**Deliverables:** A VW port to webassembly, including a simple training and inferencing JS API and a sample webpage using it.

**Mentors:** Rodrigo / Alexey
 
## Implement a local loop
**Description:** A local RL loop would make it easy for users to experiment with RL.

**Deliverables:** A binary with local join/train + changes to rl client lib.

**Mentors:** Rajan / Alexey

## Web app local loop companion
**Description:** A great local loop requires the user being able to see what's going on there. This project is to deliver a simple web app to be run locally that would provide some insights into what's going on in the local loop plus customize/configure it.

**Deliverables:** A web app packaged that can be run locally and do the above.

**Mentors:** Jacob

## Raspberry Pi port of VW
**Description:** Offer binaries/tools/docs/?? to make it amazing to work with VW on rpy.

**Deliverables:** Rasbian packages and instructions.

**Mentors:** Yann

## Alternative configuration/serialization mechanism for reductions stack
**Description:** The reduction stack Is currently defined as a series of command line arguments. Make it possible to load/save it using an alternative format such as flatbuffers.

**Deliverables:** FB-base load/save of reduction stacks

**Mentors:** Jack / Alexey

## Jupyter Notebook integration
**Description:** Jupyter notebooks is an amazing learning and code exploration platform. It should be possible to use VW with it.

**Deliverables:** Notebooks for training, learning, offline eval and ??

**Mentors:** Jacob / Alexey

## GPU backend for base learner / reductions
**Description:** GPUs are cool and fast. We should explore what would it take to have VW exploit them.

**Deliverables:** A OpenCL implementation of at least GD and maybe some of the simpler reductions.

**Mentors:** Jack / Rodrigo

## Mobile VW
**Description:** ML is not restricted to big servers, we should port VW to run on Android or iOS and write some samples showing all of it awesomeness.

**Deliverables:** VW ported to either platforms plus a sample app consuming a VW model trained on desktop to make some predictions.

**Mentors:** Alexey