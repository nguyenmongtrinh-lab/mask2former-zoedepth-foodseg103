
## 📐 System Architecture

Below is the end-to-end multi-modal architecture implemented in this repository, mapping an input 2D food image directly to its granular physical mass (Grams).

<img width="1989" height="1056" alt="image" src="https://github.com/user-attachments/assets/d7faff86-dfdf-4347-8baa-26db7d9de209" />

The processing pipeline is executed sequentially through three core stages as illustrated above:

### 1. Multi-Modal Feature Extraction & Parsing
* **Input Stage:** The system ingests a raw $512 \times 512 \times 3$ RGB food image representing a real-world dining scenario.
* **Dual-Path Processing:** * **Mask2Former Path:** The image is processed by a hierarchical Swin Transformer backbone [2] and decoded through a Masked Attention mechanism to extract $N$ discrete object queries, generating precise **2D Semantic Segmentation Masks** across 103 ingredient categories.
  * **ZoeDepth Path:** Simultaneously, the image is passed through the ZoeDepth framework [18]. By leveraging its specialized *Metric Bins* module, it eliminates scale ambiguity and generates a pixel-aligned **Metric Depth Map**, where each pixel value explicitly defines the physical distance (in meters) from the lens.

### 2. Geometry-Aware Volumetric Integration
Once the 2D masks ($\mathcal{M}$) and Metric Depth maps ($d_p$) are aligned, the system invokes a spatial computing layer:
* **Reference Plane Fitting:** The flat surface of the dining plate is computed as a reference baseline ($d_{ref}$).
* **Discrete Tifch phân (Integration):** The localized volume ($V$) for each identified food entity is calculated by aggregating the height differentials over the boundary manifold:
  $$V = \sum_{p \in \mathcal{M}} (d_{ref} - d_p) \cdot A_{pixel}$$
  Where $A_{pixel}$ represents the dynamic physical surface area of a single pixel derived from the camera's intrinsic focal parameters ($f_x, f_y$).

### 3. Density Mapping & Mass Estimation
* **Volume-to-Mass Transformation:** The computed volume tensors ($cm^3$) are multiplied by their corresponding ingredient density constants ($\rho$) fetched dynamically from a predefined **Density Lookup Table**:
  $$m = V \cdot \rho$$
* **Final Deployment:** The computed gram weights are packaged into a structured data payload and rendered in real-time on the **Gradio UI** web space for end-user interaction.
