---
widget: slider
headless: true  # This file represents a page section.

# Activate this widget? true/false
active: true

# Order that this section appears on the page.
weight: 15

# ... Put Your Section Options Here (section position etc.) ...

# Slide interval.
# Use `false` to disable animation or enter a time in ms, e.g. `5000` (5s).
interval: 5000

# Minimum slide height.
# Specify a height to ensure a consistent height for each slide.
# height: 900px; background-position:center; background-repeat: no-repeat; background-size: cover'
height: 350px
# width: 100px

# height: 300px


item:
  - title: Functional Data analysis
    content: 'My latest paper on basis selection using ML'
    # Choose `center`, `left`, or `right` alignment.
    align: left
    # Overlay a color or image (optional).
    #   Deactivate an option by commenting out the line, prefixing it with `#`.
    overlay_color: '#666'  # An HTML color value.
    # overlay_img: slider/MFG.jpg  # Image path relative to your `assets/media/` folder
    # overlay_img: slider/carousel_ml.jpg  # Image path relative to your `assets/media/` folder
    # overlay_img: slider/Hiba_eign_splits.png  # Image path relative to your `assets/media/` folder
    # overlay_img: slider/DDK_paper_curve.png  # Image path relative to your `assets/media/` folder
    overlay_img: slider/FDA_splinet_copy.jpg  # Image path relative to your `assets/media/` folder
    overlay_filter: 0  # Darken the image. Value in range 0-1.
    # Call to action button (optional).
    #   Activate the button by specifying a URL and button label below.
    #   Deactivate by commenting out parameters, prefixing lines with `#`.
    cta_label:
    cta_url: 'https://ranibasna.netlify.app/project/data_driven_kont/'
    cta_icon_pack: fas
    cta_icon: graduation-cap
  - title:
    content:
    align: right
    # overlay_color: '#555'
    # overlay_img: slider/Reproducible_research.jpg  # Image path relative to your `assets/media/` folder
    overlay_img: slider/ReproducibleResearch_2.png  # Image path relative to your `assets/media/` folder
    overlay_filter: 0.0
    cta_url: 'https://ranibasna.github.io/Reproducible-research-with-R-/'
    cta_icon_pack: fas
    cta_icon: graduation-cap
  - title: Mean Field Games
    content: 'My PhD thesis'
    align: left
    overlay_color: '#333'
    overlay_img: slider/MFG.jpg  # Image path relative to your `assets/media/` folder
    # overlay_img: slider/Hiba_eign_splits.png  # Image path relative to your `assets/media/` folder
    overlay_filter: 0.0
    cta_url: 'https://ranibasna.netlify.app/project/mean_field_games/'
    cta_icon_pack: fas
    cta_icon: graduation-cap
  - title:  UFT
    content: 'An R package for Numerical Transformation. Convert your categorical data to numerical using this package'
    align: left
    # overlay_color: '#666'
    overlay_color: '#555'
    overlay_img: slider/UFT_package.png  # Image path relative to your `assets/media/` folder
    overlay_filter: 0.2
    cta_label: see the package page
    cta_url: https://ranibasna.github.io/NumericalTransformation/
    cta_icon_pack: fas
    # cta_icon: r-project
    cta_icon: atom

  - title:  DDK
    content: 'An R package for Data-Driven Orthonormal Basis Selection using Machine Learning'
    align: right
    overlay_color: '#666'  # An HTML color value.
    # overlay_img: slider/DDK_package.png
    overlay_img: slider/DDK_FD.png
    overlay_filter: 0.2
    cta_label:
    cta_url: https://github.com/ranibasna/ddk
    cta_icon_pack: fas
    cta_icon: code
---
