<div align="center">

# DeepSpeed-FastGen: Introducing Mixtral and Phi-2 support, performance boost, and stability and software enhancements (TODO: Placeholder title)

</div>

# TODO: Add image here
<div align="center">
  <img src="" alt="" width="850"/>
</div>

# Table of Contents
1. [Introduction](#introduction)
2. [Added Model Support](#added-model-support)
3. [Performance Optimizations](#performance-optimizations)
    - [Performance Evaluation](#performance-evaluation)
4. [Stability and Software Enhancements](#stability-and-software-enhancements)
5. [Try Out DeepSpeed-FastGen](#try-out-deepspeed-fastgen)


# 1. Introduction <a name="introduction"></a>

DeepSpeed-Fastgen is an inference system framework that enables easy, fast, and affordable inference for large language models (LLMs). From general chat models to document summarization, and from autonomous driving to copilots at every layer of the software stack, the demand to deploy and serve these models at scale has skyrocketed. DeepSpeed-Fastgen utilizes the Dynamic SplitFuse technique to tackle the unique challenges of serving these applications and offer higher effective throughput than other state-of-the-art systems like vLLM.

Today, we are happy to share that we are improving DeepSpeed-Fastgen along three areas: i) greater model support, ii) performance optimizations, and iii) stability and software enhancements:
- **Support for new models**

  We introduce system support for Mixtral, Falcon, Phi-2, and Qwen models in DeepSpeed-Fastgen. Our inference optimizations for these models provide up to a.bX improvement in latency and a.bX improvement in effective throughput over other state-of-the-art frameworks like vLLM.

- **Performance Optimizations**

  [TODO Masahiro should add some text about performance optimizations here]. We demonstrate the performance optimizations with benchmarks and evaluation of DeepSpeed-Fastgen against vLLM for the newly added model support. The benchmark results can be seen in the [Performance Evaluation](#performance-evaluation) sub-section and the benchmark code is available in the [DeepSpeedExamples](https://github.com/microsoft/DeepSpeedExamples/tree/master/benchmarks/inference/mii) repository.

- **Stability and Software Enhancements**

  DeepSpeed-Fastgen contains a rich set of features for running inference with many different model families and over 20,000 HuggingFace hosted models. We extend this feature set for all models to include a RESTful API, more generation options, and support for models using the safetensor checkpoint format. Additionally, we improve on overall stability and address bugs in our original DeepSpeed-Fastgen release.

We now dive into the details of our new model support, performance optimizations, and software improvements. If you would like to get started right away with trying DeepSpeed-Fastgen, please see [Try Out DeepSpeed-FastGen](#try-out-deepspeed-fastgen).

# 2. Added Model Support <a name="added-model-support"></a>

Support for the following models has been added to DeepSpeed-FastGen.

## Mixtral

Mixtral model, a language model based on sparse mixture of experts, has demonstrated promising performance across multiple benchmarks. The Mixtral model operates by applying a router network at each layer for every token, selecting two distinct experts for processing the current state and combine their outputs. This process is dynamic, with the possibility of different experts being chosen at each timestep. This architecture ensures that while each token is exposed to a broad spectrum of parameters, it actively utilizes only a specific subset during inference.

In this release, we are pleased to announce the support for Mixtral model. We've enhanced our FastGen codebase by the integration of the Mixtral model implementation, refinements to our high-performance kernel set for efficient top-k gating, and updates to Rotary Positional Encoding (RoPE) implementation. These advancements ensure that users can fully exploit the capabilities of DeepSpeed-FastGen for executing Mixtral model inference, thereby achieving heightened performance and efficiency.

## Phi-2

Microsoft Research has introduced a suite of small language models (SLMs) named "Phi," notable for their exceptional performance across a spectrum of benchmarks. The latest addition to this suite, [Phi-2](https://www.microsoft.com/en-us/research/blog/phi-2-the-surprising-power-of-small-language-models/), is a language model boasting 2.7 billion parameters. It stands out as a testament to outstanding reasoning and language understanding capabilities, exemplifying state-of-the-art performance within the realm of base language models featuring fewer than 13 billion parameters. Notably, Phi-2 achieves parity with or surpasses models up to 25 times its size on complex benchmarks, a feat attributed to pioneering innovations in model scaling and meticulous training data curation.

Owing to its compact size, Phi-2 emerges as an ideal model for both researchers and deployment scenarios, promising a reduction in inference costs. In this DeepSpeed-FastGen release, the incorporation of partial RoPE support into our kernels signifies a noteworthy advancement. We are pleased to announce that this enhancement facilitates Phi-2 inference with DeepSpeed-FastGen.

## Falcon

Falcon is a family of large language models (LLMs) developed by the Technology Innovation Institute (TII). The Falcon models include Falcon 7B, Falcon-40B and its larger counterpart, Falcon-180B, the largest openly available language model to date. The Falcon models are optimized for inference and can be used for various applications such as chatbots, question answering systems, and more.

A closer examination of the architectural nuances within the Falcon series reveals notable distinctions. Specifically, the Falcon 7B model diverges slightly from Falcon-40B; notably, Falcon-40B incorporates an additional layer norm preceding the parallel MLP layer, a feature absent in the Falcon 7B model. In contrast, Falcon-180B adheres to the same architecture as Falcon-40B but stands out as a scaled-up version.
In this DeepSpeed-FastGen release, we are pleased to announce the addition of support for all Falcon model sizes.

# 3. Performance Optimizations <a name="performance-optimizations"></a>

[TODO Rework this section]

Comparison for several model architectures: 
  - **For general:** For long prompts and short sequence generation, we can fully utilize the benefits of SplitFuse, which combines prompt processing and decoding (token generation) in a single forward pass. This provides a significant advantage over vLLM in these scenarios. For short prompts and long sequence generation, where most forward passes run purely for decoding, our highly optimized engine and the efficient scheduler for ragged batching demonstrate impressive performance. 
  - **Falcon:** Given the model's substantial size, the majority of computations are dedicated to forward passes, while the overhead of scheduling and token sampling is relatively minor. The performance difference is primarily attributed to our highly optimized engine designed for inference. 
  - **Mixtral:** We developed a new MoE module, which contains kernels optimized for our inference engine. The enhancements in the decoding phase, included in this release, significantly improve throughput and efficiency in generating long sequences. 

System optimization in this release: SplitFuse effectively enhances utilization by simultaneously computing prompts and decoding (generating tokens). However, we observed a significant overhead for scheduling ragged batching, especially when generating long sequences from numerous concurrent requests. In this release, we've minimized this scheduling overhead for querying KV cache states. As a result, there's a notable improvement in the performance of long sequence generation. 

## Performance Evaluation

NOTE: DO NOT USE PIPELINE FOR BENCHMARKS - address vLLM benchmarks

### Mixtral

DeepSpeed requires less memory than vLLM, able to run on 2xA6000

TODO Which plots to show?

### Falcon

TODO Which plots to show?

### Phi-2

TODO Which plots to show?

<div align="center">
  <img src="assets/images/th_lat_curve_phi-2_tp1_p1200g60.png" width="640">

  *Figure X:*

</div>

<div align="center">
  <img src="assets/images/th_lat_curve_phi-2_tp1_p1200g128.png" width="640">

  *Figure X:*

</div>

<div align="center">
  <img src="assets/images/th_lat_curve_phi-2_tp1_p1900g60.png" width="640">

  *Figure X:*

</div>

<div align="center">
  <img src="assets/images/th_lat_curve_phi-2_tp1_p1900g128.png" width="640">

  *Figure X:*

</div>

# 4. Stability and Software Enhancements <a name="stability-and-software-enhancements"></a>

TODO: Should we convert this into a bulleted list?

## Support for safetensor checkpoints
Some HuggingFace-hosted model checkpoint weights are provided only in the safetensor format. We extend our HuggingFace checkpoint engine to work with the safetensor format to support even more models!

See [PR-4659](https://github.com/microsoft/DeepSpeed/pull/4659), [PR-296](https://github.com/microsoft/DeepSpeed-MII/pull/296) for more details.

## Added RESTful API

We add the option to automatically stand up a RESTful API when creating DeepSpeed-Fastgen persistent deployments in DeepSpeed-MII. This API provides a way for users to send prompts to their deployments and receive responses using HTTP POST methods and tools like `curl` or python's `request` package. The RESTful API provides the same high throughput and low latency performance as our python APIs. For more information, please see [TODO Add link to MII RESTful API README section].

See [PR-348](https://github.com/microsoft/DeepSpeed-MII/pull/348), [PR-328](https://github.com/microsoft/DeepSpeed-MII/pull/328), [PR-294](https://github.com/microsoft/DeepSpeed-MII/pull/294) for more details.

## Added deployment and generate options

We extend the customizability of DeepSpeed-Fastgen deployments and text-generation. Users can now specify a `device_map` when creating non-persistent pipelines and persistent deployments that controls which GPUs to use for hosting a model. Additionally, the interfaces between pipelines and deployments now match and include options for setting top-p, top-k, and temperature values. For additional information about the user-exposed options, please see [TODO links to pipeline and deployment sections in MII README].

See [PR-331](https://github.com/microsoft/DeepSpeed-MII/pull/331), [PR-280](https://github.com/microsoft/DeepSpeed-MII/pull/280), [PR-275](https://github.com/microsoft/DeepSpeed-MII/pull/275), [PR-268](https://github.com/microsoft/DeepSpeed-MII/pull/268), [PR-295](https://github.com/microsoft/DeepSpeed-MII/pull/295), for more details.

## Mitigate risk of deadlock

In use-cases where many prompts are sent to a deployment in a small time window, deadlock can occur in the DeepSpeed-Fastgen inference engine where no text-generation progress is made on any prompts. We have made changes to mitigate this situation and continue text-generation. While not completely resolved, we continue to investigate a fix for these situations that arrive when the deployment is under heavy load. [TODO Masahiro can probably provide more information here]

See [PR-274](https://github.com/microsoft/DeepSpeed-MII/pull/274) for more details.

## Inference Checkpoints

We add the capability to create inference engine snapshots to DeepSpeed-Fastgen. This reduces the loading time for large models in future deployments.

See [PR-4664](https://github.com/microsoft/DeepSpeed/pull/4664) for more details.

## General stability and bug fixes

We include many bug fixes and stability improvements to DeepSpeed-Fastgen. This includes fixing issues with some OPT model size variants, bugs with MII configuration options, and improved error messages.

See [PR-4938](https://github.com/microsoft/DeepSpeed/pull/4938), [PR-4920](https://github.com/microsoft/DeepSpeed/pull/4920), [PR-4739](https://github.com/microsoft/DeepSpeed/pull/4739), [PR-4694](https://github.com/microsoft/DeepSpeed/pull/4694), [PR-4634](https://github.com/microsoft/DeepSpeed/pull/4634), [PR-367](https://github.com/microsoft/DeepSpeed-MII/pull/367), [PR-350](https://github.com/microsoft/DeepSpeed-MII/pull/350), for more details.

# 5. Try Out DeepSpeed-FastGen <a name="try-out-deepspeed-fastgen"></a>

We are very excited to share this DeepSpeed-FastGen release.

* To get started, please visit our GitHub page for DeepSpeed-MII: [GitHub Landing Page](https://github.com/microsoft/DeepSpeed-MII)

DeepSpeed-FastGen is part of the bigger DeepSpeed ecosystem comprising a multitude of Deep Learning systems and modeling technologies. To learn more,

* Please visit our [website](https://www.deepspeed.ai/) for detailed blog posts, tutorials, and helpful documentation.
* You can also follow us on our [English Twitter](https://twitter.com/MSFTDeepSpeed), [Japanese Twitter](https://twitter.com/MSFTDeepSpeedJP), and [Chinese Zhihu](https://www.zhihu.com/people/deepspeed) for latest news on DeepSpeed.

DeepSpeed welcomes your contributions! We encourage you to report issues, contribute PRs, and join discussions on the [DeepSpeed GitHub](https://github.com/microsoft/DeepSpeed/) page. Please see our [contributing guide](https://github.com/microsoft/DeepSpeed/blob/master/CONTRIBUTING.md) for more details. We are open to collaborations with universities, research labs, and companies, such as those working together on deep learning research, applying DeepSpeed to empower real-world AI models and applications, and so on. For such requests (and other requests unsuitable for GitHub), please directly email to deepspeed-info@microsoft.com.

The following items are on our roadmap and we plan to engage with our community on these through our GitHub issues and PRs:

TODO Update roadmap items

**"Star" our [DeepSpeed GitHub](https://github.com/microsoft/DeepSpeed/) and [DeepSpeedMII GitHub](https://github.com/microsoft/DeepSpeed-MII/) repositories if you like our work!**