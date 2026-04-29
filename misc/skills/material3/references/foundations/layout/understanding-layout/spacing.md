---
source_url: https://m3.material.io/foundations/layout/understanding-layout/spacing
fetched_at: 2026-04-29
section: foundations
---

Layout basics
Layout is the visual arrangement of elements on the screen
Overview
Spacing
Parts of layout
Density
Hardware considerations
Bidirectionality & RTL
On this page
Layout basics
Mirroring
Text rendering
Icons and symbols
Time
Canonical layout examples
Component examples
Swipe gestures
On this page
Layout basics
Overview
Information density
Component scaling
Targets
Pixel density
On this page
Layout basics
Display cutout
Foldable devices
Multi-window mode
On this page
Layout basics
What’s new
Layout terms
On this page
Layout basics
Navigation region
Body region
Panes
How panes adapt
App bars
Columns
Drag handle
On this page
Layout basics
Grouping
Margins
Spacers
Padding

Grouping

Grouping is a method for connecting related elements that share a context, such as an image grouped with a caption. It visually relates elements and establishes boundaries to differentiate unrelated elements.

By placing a caption under an image this composition shows an explicit group

Explicit grouping uses visual boundaries such as outlines, dividers, and shadows to group related elements in an enclosed area. Explicit grouping can also indicate that an item is interactive, such as list items contained between dividers, or a card displaying an image and its caption.

The elements in this card are explicitly grouped

Implicit grouping uses close proximity and open space (rather than lines and shadows) to group related items. For example, a headline closely followed by a subhead and thumbnail image are implicitly grouped together by proximity and separated from other headline-subhead-thumbnail groups by open space.

Images in a carousel are grouped by their proximity

Margins

Margins are the spaces between the edge of a window area and the elements within that window area.

Margin widths are defined using fixed or scaling values for each window size class. To better adapt to the window, the margin width can change at different breakpoints. Wider margins are more appropriate for larger screens, as they create more open space around the perimeter of content.

See margin measurements for each window class:  compact, medium, expanded, large, and extra-large.

A margin separates the edge of the screen from the elements on the screen

Spacers

A spacer refers to the space between two panes in a layout. Spacers measure 24dp wide.

A spacer splits two panes from each other

A spacer can contain a drag handle that adjusts the size and layout of the panes. The handle's touch target slightly overlaps the panes.

Drag handle touch target

Padding

Padding refers to the space between UI elements. Padding can be measured vertically and horizontally and does not need to span the entire height or width of a layout. Padding is measured in increments of 4dp.

Padding separates a headline from a image above
