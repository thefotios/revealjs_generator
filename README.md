# Purpose
This project is aimed to replicate some of the functionality of showoff,
but using reveal.js

# Usage
1. Update your submodules
  `git submodule update --init`
1. Create slides in the slides folder
  * They can be formatted as anything [Tilt] understands
    * They must also be named properly
    * Additional gems may be required
  * Slides will be included in sorted order
1. Run `rake` to compile your slides and then updload them to GitHub
   pages
1. Slides will now be viewable at http://username.github.com/project

[Tilt]: https://github.com/rtomayko/tilt