# Multilingual OCR Enhancement via Synthetic Data: Qianfan-OCR Covering 196 Languages

## Background

The growing demand for globalized applications poses significant challenges to the multilingual coverage of OCR models. Traditional OCR systems tend to perform reliably only on a handful of dominant languages, while struggling with low-resource languages, complex scripts, and non-standard writing systems. Many low-resource languages inherently lack high-quality annotated data, and manually labeling real OCR data is prohibitively expensive, making it infeasible to scale language coverage through human annotation alone.

The community has explored synthetic data extensively: SynthText [1] pioneered natural scene text synthesis; TRDG (TextRecognitionDataGenerator) [2] became a widely adopted open-source tool for OCR data augmentation; SynthTIGER [3] further improved the realism and diversity of synthesized text. More recently, diffusion-based approaches such as TextDiffuser [4] and TextFlux [5] have explored generating text images directly via generative models, showing promise in style transfer. However, most of these works focus on English or a small set of dominant languages, and lack systematic support for 196-language-scale coverage and correct rendering of complex writing systems.

Building on existing synthesis paradigms, our work focuses on two core challenges in multilingual settings: font renderability validation and script-aware differential rendering. Starting from high-quality multilingual plain text, we automatically synthesize images resembling real document scenes through controllable rendering and perturbation modeling, while naturally obtaining precise text-image alignment annotations. Through this pipeline, as presented below, Qianfan-OCR's multilingual capability has been expanded to cover 196 languages.

```mermaid
%%{init: {'theme': 'default', 'themeVariables': {'fontSize': '11px'}, 'flowchart': {'nodeSpacing': 25, 'rankSpacing': 35}}}%%
flowchart TD
    A[("📚 Multilingual Plain Text Corpus\n196 Languages")]

    A --> B

    subgraph PREP ["① Data Preparation"]
        B["Font Renderability Filtering"]
    end

    B --> C

    subgraph RENDER ["② Differential Rendering Engine"]
        C{"Writing System Detection"}
        C -->|"LTR\nLatin / Cyrillic / Greek…"| D["Left-aligned Layout\nStandard Text Rendering"]
        C -->|"RTL\nArabic / Hebrew…"| E["Right-aligned Layout\nWord Order Reversal"]
        E -->|"Arabic"| F["Arabic Reshaping\nLigature Reordering"]
        E -->|"Hebrew etc."| G["Direct RTL Layout"]
    end

    D & F & G --> H

    subgraph AUG ["③ Visual Diversity Augmentation"]
        H["Random Perturbation: Layout, Font, Texture, Stains…"]
    end

    H --> I

    subgraph ENG ["④ Engineered Parallel Scheduling"]
        I["Multi-machine Concurrency · Checkpoint Resume"]
    end

    I --> J[("🖼️  Synthesized Images\n+ Precise Text-Image Alignment Annotations")]
    J --> K["Model Pre-training"]
    K --> L["✅ Qianfan-OCR\n196 Languages Covered"]

    style PREP fill:#EFF6FF,stroke:#3B82F6
    style RENDER fill:#F0FDF4,stroke:#22C55E
    style AUG fill:#FFF7ED,stroke:#F97316
    style ENG fill:#FDF4FF,stroke:#A855F7
    style A fill:#DBEAFE,stroke:#2563EB
    style J fill:#DCFCE7,stroke:#16A34A
    style L fill:#F0FDF4,stroke:#15803D,color:#15803D
```

## 1. Character Filtering and Differential Rendering for Correctness

When synthesizing images from text, an appropriate font must be selected for each language. However, individual characters may still fall outside a font's supported glyph set, leading to rendering failures and annotation errors. To address this, we introduce a dedicated font renderability filtering stage during data preparation:

- Use `fonttools` to pre-generate the supported character set for each font and persist it as a cache
- Before synthesis, query the cache first to quickly validate whether the target text is fully covered by the chosen font
- For cache misses, dynamically generate the character set and write it back to the cache
- As a fallback for edge cases, perform per-character rendering validation using `Pillow`

Beyond font coverage, different writing systems cannot be handled by a single unified rendering pipeline. Arabic, for example, is written right-to-left and involves context-dependent glyph shaping — rendering it with standard text drawing produces visually incorrect output. Our differential rendering adaptation addresses this with the following key strategies:

- Automatically detect RTL languages and activate right-to-left layout logic
- Apply character reshaping (Arabic Reshaping) for scripts requiring ligature reordering, ensuring visually correct connected forms
- Use word-width-aware automatic line wrapping and alignment to ensure stable multi-line text layout

Together, these two strategies significantly improve the robustness and correctness of the multilingual reverse synthesis pipeline across diverse texts and fonts. Synthesis results across different languages are shown below.

<img src="images/case_1.png" width="24%"/><img src="images/case_2.png" width="24%"/><img src="images/case_3.png" width="24%"/><img src="images/case_4.png" width="24%"/>

## 2. Visual Diversity for Enhanced Realism

To bridge the gap between simple synthetic images and the real-world inputs models encounter in production, we incorporate diverse layout and visual modeling during image synthesis, including:

- Random sampling of font families and sizes to increase typographic diversity
- Random perturbation of margins, line spacing, word spacing, and page dimensions
- Simulation of document structures such as multi-column layouts, page spreads, and page edges
- Random ink blots, stains, and slight font distortions
- Texture background overlays to enhance the look and feel of physical printed documents

These designs bring the training data closer to real-world document distributions, improving the model's generalization to varied document structures and layout changes.

**Font Diversity**: To help the OCR model learn to recognize characters themselves, rather than memorizing pixel patterns of a specific font, we randomly sample fonts of different styles during synthesis. The figures below show the same Latin text rendered in four different font styles.

<img src="images/case_5.png" width="24%"/><img src="images/case_6.png" width="24%"/><img src="images/case_7.png" width="24%"/><img src="images/case_8.png" width="24%"/>

**Layout Diversity**: Real historical documents come in a wide variety of layouts — single-column, double-column, and open book spreads, among others. To cover these scenarios, we introduce layout randomization during synthesis.

<img src="images/case_9.png" width="24%"/><img src="images/case_10.png" width="24%"/><img src="images/case_11.png" width="24%"/><img src="images/case_12.png" width="24%"/>

**Visual Perturbation**: Real-world document images often carry various forms of visual noise — scanner textures, paper tones, print quality variations, and age-related degradation. To ensure the synthetic data covers these real distributions, we design multi-level visual perturbation.

<img src="images/case_13.png" width="24%"/><img src="images/case_14.png" width="24%"/><img src="images/case_15.png" width="24%"/><img src="images/case_16.png" width="24%"/>

## Conclusion and Future Work

Pre-training Qianfan-OCR on the constructed multilingual synthetic data has significantly expanded language coverage; recognition accuracy on low-resource languages has improved substantially; robustness under complex fonts and noisy conditions has been enhanced; and overall performance in multilingual mixed scenarios is more stable.

The current approach still has limitations: at the visual modeling level, each image is primarily composed of a single language, and document-level effects such as perspective distortion, lighting variation, and margin annotations are not yet covered; at the scenario coverage level, the focus remains on printed documents, with handwritten text, scene text, and logo-style text still posing challenges.

Looking ahead, this framework can evolve in several directions:
- Incorporate complex layout design capabilities from DocSynth [6] to further enrich document structure modeling
- Draw from language-specific synthesis approaches such as SARD [7] to migrate and integrate their language-specialized augmentation strategies into the multilingual framework
- Introduce perspective transforms and lighting perturbations as additional visual augmentations, and explore multi-font mixed rendering and handwriting style synthesis to cover a broader range of application scenarios

Through the paradigm of reverse synthesis withr engineered production, it is possible to continuously and cost-effectively scale the data foundation for multilingual OCR, providing support for document intelligence in global deployment scenarios.

## References

[1] A. Gupta, A. Vedaldi, and A. Zisserman, "Synthetic data for text localisation in natural images," in *Proceedings of the IEEE Conference on Computer Vision and Pattern Recognition (CVPR)*, 2016, pp. 2315–2324.

[2] G. Belval, TextRecognitionDataGenerator, 2018. [Online]. Available: https://github.com/Belval/TextRecognitionDataGenerator

[3] M. Yim et al., "SynthTIGER: Synthetic text image generator towards better text recognition models," in *Proceedings of the International Conference on Document Analysis and Recognition Workshops (ICDARW)*, 2021, pp. 109–124.

[4] M. Chen et al., "TextDiffuser: Diffusion models as text painters," *arXiv preprint* arXiv:2305.10855, 2023.

[5] Y. Xie et al., "TextFlux: An OCR-Free DiT model for high-fidelity multilingual scene text synthesis," *arXiv preprint* arXiv:2505.17778, 2025.

[6] S. Biswas, P. Riba, J. Lladós, and U. Pal, "DocSynth: A layout guided approach for controllable document image synthesis," in *Proceedings of the International Conference on Document Analysis and Recognition (ICDAR)*, 2021.

[7] O. Nacar et al., "SARD: A large-scale synthetic Arabic OCR dataset for book-style text recognition," *arXiv preprint* arXiv:2505.24600, 2025.
