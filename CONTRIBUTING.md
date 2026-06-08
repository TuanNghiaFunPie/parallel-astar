# Contributing

Thank you for your interest in contributing to the Parallel A* Pathfinding project!

## Getting Started

1. **Fork** the repository
2. **Clone** your fork: `git clone https://github.com/<your-username>/parallel-astar.git`
3. **Create a branch** for your feature: `git checkout -b feature/your-feature-name`

## Development Setup

```bash
# Create virtual environment
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt

# For CUDA development, ensure CUDA Toolkit is installed
# Visit: https://developer.nvidia.com/cuda-downloads
```

## Code Style

- **Python**: Follow PEP 8
- **C++/CUDA**: Use consistent formatting (spaces, 100-char line limit)
- **Comments**: Write clear, English comments explaining the "why", not just the "what"
- **Docstrings**: Use Python docstrings for modules and functions

## Types of Contributions

### 1. Bug Reports
- Open an issue with detailed description
- Include: OS, GPU model, Python version, error messages
- Provide minimal reproducible example if possible

### 2. Feature Requests
- Suggest improvements that align with project scope
- Include use cases and expected benefits

### 3. Code Improvements
- **Performance optimizations**: Benchmark before/after
- **Portability fixes**: Test on multiple platforms if possible
- **Documentation**: Clarify existing docs, add missing explanations
- **Refactoring**: Improve code structure while maintaining functionality

### 4. Benchmark Extensions
- Add new graph sizes/query distributions
- Test different GPU architectures
- Analyze scalability trends

## Pull Request Process

1. **Ensure your code passes validation:**
   ```bash
   # For Python, check for syntax errors
   python -m py_compile *.py
   ```

2. **Update documentation** if you modify functionality

3. **Provide clear PR description:**
   - What problem does it solve?
   - How was it tested?
   - Any breaking changes?

4. **Link related issues** (if any)

## Project Scope

**In scope:**
- A* algorithm improvements
- CUDA optimization techniques
- Benchmark extensions
- Documentation improvements
- Portability across platforms

**Out of scope:**
- Alternative pathfinding algorithms (Dijkstra, Bellman-Ford)
- Map rendering/visualization frameworks
- Non-GPU implementations

## Questions?

- Check README.md for general information
- Review existing code comments for implementation details
- Open a discussion issue for architectural questions

## License

By contributing, you agree that your contributions will be licensed under the MIT License.

---

**Thank you for making this project better! 🚀**
