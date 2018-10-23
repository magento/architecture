# Minimum Viable Features for Fotorama Replacement

**Note**: This should not be an exhaustive list of every feature a slider should have. Instead, this list should represent the minimal number of features needed with a basic ecomm site. Those seeking something more powerful will still be able to write a module that swaps out the chosen lib with something that better suits their needs.

- Supports touch actions (swiping) on mobile devices (not required on desktop)
- Responsive
- Supports pre-rendering image tags in the DOM (slider should not be the one injecting `<img />` tags)
- [Accessible](https://www.w3.org/WAI/tutorials/carousels/). Basic requirements:
    - Pausable
    - Keyboard support
    - Proper usage of markup and aria roles