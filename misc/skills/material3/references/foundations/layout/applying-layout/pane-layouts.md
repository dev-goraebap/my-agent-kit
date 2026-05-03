---
source_url: https://m3.material.io/foundations/layout/applying-layout/pane-layouts
fetched_at: 2026-04-29
section: foundations
---

Applying layout
Window size classes help create layouts that scale across devices of all shapes and sizes
Pane layouts
Window size classes
Compact
Medium
Expanded
Large & extra-large
On this page
Applying layout
Navigation
Body region
Spacing
Special considerations
On this page
Applying layout
Navigation
Body pane
Spacing
Special considerations
On this page
Applying layout
Navigation
Body pane
Additional panes
Spacing
Special considerations
On this page
Applying layout
Navigation
Body region
Spacing
Special considerations
On this page
Applying layout
Choosing a pane layout
Pane expansion & resizing
Displaying multiple panes
How panes adapt
Spatial panels
Accessibility considerations

Choosing a pane layout

All layouts are made up of 1–3 panes. The type of layout and amount of panes you choose should depend on the window size class and the type of product being built.

Window size	Recommended pane total	Other pane totals
Compact	1	--
Medium	1	2
Expanded	2	1
Large	2	1
Extra-large	2	1, 3

Panes can be:

Fixed: Width doesn’t change based on available space
Flexible: Responsive to available space, and can grow and shrink

All layouts need at least one flexible pane. 

Fixed pane
Flexible pane

Panes can be permanent or temporary. Temporary panes can appear and be dismissed when necessary, affecting the layout and size of other panes.

Panes can be displayed permanently side by side

Temporary panes can be dismissed

Single-pane layouts

Single-pane layouts use one flexible pane that extends to the available space in a layout’s width. They can be used in any window size, but are recommended for compact and medium window sizes.

A single flexible pane adapts to any window size

Two-pane layouts
Split-pane layout

A split-pane layout keeps the spacer visually centered. It’s best for foldable devices and dynamic layouts.

When a navigation rail or drawer is present, it only reduces the size of one pane. The other pane remains at 50% of the window width.

The navigation and first pane are 50% of the window width to keep the spacer visually centered

With a navigation bar, or no navigation, both panes span 50% of the window width by default.

With no navigation rail visible, split-pane layouts set each pane to 50% width by default

Fixed and flexible layout

This layout is common for expanded, large, and extra-large windows. The fixed and flexible panes can appear in whichever order is best for the content.

The fixed pane is often temporary, and used for side sheets or lists with light information density.

Fixed pane
Flexible pane

Three-pane layouts

While less common, the extra-large window size class supports using a standard side sheet as a third pane. When the side sheet is present, the navigation drawer can remain visible, collapse into a navigation rail, or hide completely. Don't use more than three panes. 

Note: Fixed panes in this window size are recommended to be 412dp, but side sheets have a default maximum width of 400dp.

Standard side sheet as a third pane

Pane expansion & resizing

Panes can be resized, expanded, and collapsed using drag handles. 

In split-pane layouts, both flexible panes can be freely adjusted, or can snap to certain widths.
In fixed and flexible layouts, the drag handle can fully collapse and expand the fixed pane. This makes it easy to switch between a single-pane and two-pane layout.

The drag handle should also toggle between layout sizes when selected. This can be a tap, double tap, or long press.

Drag handles can adjust pane size in a list-detail layout

In expanded, large, and extra-large window sizes, two-pane layouts can be customized to snap to set widths when resized.

The recommended custom widths are:

360dp
412dp
Split-pane with spacer centered visually

Panes can snap to custom widths when releasing the drag handle

Persistent pane resizing

The persistent resizing behavior remembers the user's pane width preference. Use this for most resizable layouts.

Pane widths persist even after a user closes the app

The width persists even after a window size class change. This means that if a two-pane layout is collapsed to one pane at any window size, it will remain collapsed even when changing window sizes.

When a two-pane layout is resized to a single full-width pane, that pane should remain at full-width after switching window sizes

Temporary pane resizing

The temporary resizing behavior doesn't remember user preferences for pane width. This is primarily used in supporting pane layouts where resizing is uncommon.

Supporting pane layouts can have a pane drag handle to temporarily resize the secondary content

With temporary resizing, panes should always return to the default layout after the pane or app is closed and reopened. This ensures content is a suitable size for most interactions.

The pane width can be adjusted using the drag handle

Displaying multiple panes

There are three ways that multiple panes can be displayed in a layout: co-planar, floating, or docked. Choose the method best for each window size class.

Co-planar: Panes are displayed side by side

Floating: A pane is displayed above other panes or content, like a dialog

Docked: A pane is displayed above other panes and one of its edges extends beyond one side of the screen, like a bottom sheet

How panes adapt

Pane layouts can adapt using three strategies: show and hide, levitate, or reflow. When the window is resized or changes orientation, these strategies allow panes to reorganize themselves to preserve context and meaning.

Show and hide

As the window size changes, panes can enter and exit the screen or appear next to one another.

A pane can be shown or hidden depending on the available window space

Levitate

Panes can be elevated above other content as floating or docked panes. This strategy helps panes appear relative to their triggers. 

Floating panes appear in front of the body content, and can be customized to be dragged or resized. When adding controls that resize or move a floating pane, provide accessible controls.

A co-planar pane can float when switching window size classes

On large screens, the scrim behind a floating pane is optional.

Floating pane with a scrim
Floating pane without a scrim

Docked panes are usually at the bottom of the window, like a bottom sheet.

In medium and expanded window sizes, docked panes can adapt into floating panes.

A docked pane can adapt into a floating pane

Alternatively, in medium and expanded window sizes, a docked pane can adapt into a co-planar pane. 

A docked pane can adapt into a co-planar pane

On large screens, docked panes can remain docked or become co-planar.

Docked pane
Co-planar pane

Reflow

Panes can be reorganized on screen as the window size or orientation changes, also known as reflow. For example, in a vertical orientation, the supporting pane can move underneath the primary pane.

In a vertical orientation, the supporting pane can move below the primary pane

Reflow also applies to window sizes. When there’s not enough horizontal space for panes, they can stack vertically instead.

Panes can change size, location, and orientation when switching screen sizes

Spatial panels

On XR devices, pane layouts can be presented in disconnected spatial panels. These panels must have clear containment to make them easy to see on any background.

The content in a spatial pane may use implicit grouping when the pane has an explicit container to distinguish it from the environment.

When the pane uses explicit containment, content can use implicit grouping

Accessibility considerations

Coplanar panes

For coplanar panes, the focus order matches the visual arrangement of the panes on the screen.

Floating

A modal floating pane disappears when a user interacts with something behind it. When a modal pane is active the elements behind it can’t be interacted with. When a floating pane is modal, focus moves automatically to the first element in the pane, and when the pane is closed, focus moves back to the element that triggered it, like a dialog. If the modal pane was triggered automatically, focus should still move to it, but when it is closed, focus should go to the next most logical element on the screen.
When a non-modal floating pane is open, other parts of the application can be interacted with. For non-modal panes, focus should be able to move to and from the pane, and the pane should also be available in a logical reading order of the screen.

Docked

Docked panes have the same focus requirements as modal and non-modal panes. The focus order should match the visual arrangement of panes.
