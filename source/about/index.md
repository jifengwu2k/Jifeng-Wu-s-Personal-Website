---
title: Home
date: 2025-05-20
---

I am an ECE researcher focusing on making computer systems more efficient and accessible by bridging programming languages, compilers, and hardware.

I obtained my Master's of Science in Computer Science from [**UBC**](https://www.ubc.ca/) working with [**Caroline Lemieux**](https://www.carolemieux.com/). My Master's thesis [**"QuAC: Quick Attribute-Centric Type Inference for Python"**](/static/pdfs/quac_oopsla_2024.pdf) (OOPSLA'24) implemented QuAC, a novel Python type inference tool that leverages attribute sets and information retrieval techniques to predict types.

Before that, I obtained my Bachelor of Engineering from the [**School of Computer Science, Wuhan University**](http://cs.whu.edu.cn/) under the guidance of [**Qingan Li**](http://liqingan.cn/liqingan/). My Bachelor's thesis [**"Effective Stack Wear Leveling for NVM"**](/static/pdfs/Effective_Stack_Wear_Leveling_for_NVM.pdf) (TCAD'23) proposed Loop2Recursion, a compiler-assisted stack wear leveling technique implemented as an LLVM pass for increasing the lifespan of NVM by converting wear-heavy loops into recursive functions. I have also conducted research in graph and trajectory data mining, advised by Yuanyuan Zhu.

Outside research, I enjoy blogging, contributing to open-source projects, and mentoring students.

[**Blog**](/categories/) | [**Email**](mailto:jifengwu2k@gmail.com) | [**Facebook**](https://www.facebook.com/profile.php?id=100087278086868) | [**GitHub**](https://github.com/jifengwu2k) | [**Instagram**](https://www.instagram.com/jifengwu2k/) | [**LinkedIn**](https://www.linkedin.com/in/jifeng-wu-a3b7b124a/) | [**PyPI**](https://pypi.org/user/jifengwu2k/) | [**Resume**](/static/pdfs/Resume.pdf) | [**WeChat**](/static/images/wechat_qr_code.png) | [**X**](https://x.com/jifengwu2k)

Utilities: 

- [**Clipboard to Markdown**](/static/utilities/clipboard-to-markdown.html)
- [**Webcam Viewer**](/static/utilities/webcam-viewer.html)

------

## Papers and Essays

YAML Filesystem Trees
: We introduce YAML filesystem trees, a compact, human- and machine-readable representation of file and directory trees in YAML. This notation aims to offer intuitive editing and precise semantics, facilitating diverse applications: environment specification, package manifests, deployment scripts, reproducible research, and more. We describe the format, enumerate its core properties, and provide a reference implementation for recursive traversal; we also discuss applications such as copying, validation, extraction, and visualization.

- Time: January 2026
- [**Essay**](https://zenodo.org/records/18330247)

BibTeX:

```bibtex
@misc{wu_2026_18330247,
  author       = {Wu, Jifeng},
  title        = {YAML Filesystem Trees},
  month        = jan,
  year         = 2026,
  publisher    = {Zenodo},
  doi          = {10.5281/zenodo.18330247},
  url          = {https://doi.org/10.5281/zenodo.18330247},
}
```

White Paper: A Dynamic Gray-Box Methodology for Analyzing and Simplifying PyTorch Models
: Machine learning researchers and practitioners often face challenges when analyzing, interpreting, and repurposing deep learning models, particularly those built with popular, highly dynamic frameworks such as PyTorch. Existing static analysis and tracing techniques offer limited support for the breadth of constructs found in real-world codebases. This work-in-progress proposes a dynamic, gray-box methodology for systematically analyzing PyTorch models, extracting human-interpretable module hierarchies, and automatically synthesizing minimal, skeletal module classes that preserve functional equivalence and remain compatible with the original model weights. This document outlines a general strategy, open to further extensions, for approaching these challenges in a principled manner.

- Time: January 2026
- [**Essay**](https://zenodo.org/records/18216741)

BibTeX:

```bibtex
@misc{wu_2026_18216741,
  author    = {Wu, Jifeng},
  title     = {White Paper: A Dynamic Gray-Box Methodology for Analyzing and Simplifying PyTorch Models},
  month     = jan,
  year      = 2026,
  publisher = {Zenodo},
  doi       = {10.5281/zenodo.18216741},
  url       = {https://doi.org/10.5281/zenodo.18216741},
}
```

QuAC: Quick Attribute-Centric Type Inference for Python (Master's Thesis)
: We implemented QuAC, a novel type inference tool for Python that collects attribute sets for Python expressions and uses information retrieval techniques to predict classes. Compared to baseline methods, QuAC efficiently handles rare non-builtin types and container type parameters and improves performance by an order of magnitude.

- Accepted to: OOPSLA 2024
- Mentor: Prof. Caroline Lemieux
- Time: July 2023 - August 2024
- [**Paper**](/static/pdfs/quac_oopsla_2024.pdf)
- [**GitHub Repository**](https://github.com/jifengwu2k/quac)
- [**Lessons Learned**](/2024/08/23/Lessons-learned-from-master-s-thesis/)

Effective Stack Wear Leveling for NVM (Bachelor's Thesis)
: We increase the lifespan of non-volatile memory with limited write durability by implementing an LLVM pass that can convert wear-heavy loops in programs into recursive functions.

- Published in: IEEE Transactions on Computer-Aided Design of Integrated Circuits and Systems (2023)
- Authors: Jifeng Wu, Wei Li, Libing Wu, Mengting Yuan, Chun Jason Xue, Jingling Xue, Qingan Li
- Mentor: Prof. Qingan Li
- Time: August 2021 - August 2022
- [**Paper**](/static/pdfs/Effective_Stack_Wear_Leveling_for_NVM.pdf)
- [**GitHub Repository**](https://github.com/jifengwu2k/loop2recursion)

Community Detection Using Social Relations and Trajectories
: Community detection is an essential task in social network analysis, but many friends on social networks are not close to one another in the real world. We introduce a novel approach that utilizes user trajectories to identify cohesive groups of users who frequently hang out together and presents algorithms for efficiently calculating spatiotemporal similarity between trajectories and community detection.

- Time: September 2019 - June 2021
- [**Essay**](/static/pdfs/Community_Detection_using_Social_Relations_and_Trajectories.pdf)
- [**GitHub Repository**](https://github.com/jifengwu2k/community-detection-using-social-relations-and-trajectories)

------

## Friends

- [**Zhaowei Zhang @ Peking University**](https://zowiezhang.github.io/)
- [**Bingchu Zhao @ KTH**](https://timmygiorno.github.io/)
- [**Ziqi Rong @ University of Michigan**](https://www.zqrong.com/)
- [**Jianhao Wu @ CUHK**](https://rushingfox.github.io/)
- [**Hao Wang @ HKUST**](https://figerhaowang.github.io/)
