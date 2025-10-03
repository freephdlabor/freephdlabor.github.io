# ICML 2025 LaTeX Template

This folder contains the official ICML 2025 conference paper template files.

## Contents

- `icml2025.sty` - Main style file for ICML 2025
- `icml2025.bst` - Bibliography style file for ICML 2025
- `template.tex` - Example template showing how to use the style
- `algorithm.sty` - Algorithm environment support
- `algorithmic.sty` - Algorithmic environment support

## Usage

To use this template in your paper:

1. Copy all `.sty` and `.bst` files to your paper directory
2. In your LaTeX document preamble, add:
   ```latex
   \usepackage[accepted]{icml2025}
   ```
   or for review version:
   ```latex
   \usepackage{icml2025}
   ```

3. Use the bibliography style:
   ```latex
   \bibliographystyle{icml2025}
   ```

## Note

The WriteupAgent in this codebase currently uses ICML 2024 template. To update it to use ICML 2025:
- Update `freephdlabor/toolkits/writeup/latex_generator_tool.py` line 389
- Copy the ICML 2025 files to `freephdlabor/toolkits/writeup/`

## Source

These template files were copied from:
`AI-Scientist-v2/ai_scientist/blank_icml_latex/`