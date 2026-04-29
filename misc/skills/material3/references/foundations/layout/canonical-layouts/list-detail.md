---
source_url: https://m3.material.io/foundations/layout/canonical-layouts/list-detail
fetched_at: 2026-04-29
section: foundations
---

Canonical layouts
Canonical layouts are designs for common screen layouts across all window size classes
Overview
List-detail
Supporting pane
Feed
On this page
Canonical layouts
Usage
Dividing space
Across window size classes
Behavior
On this page
Canonical layouts
Usage
Dividing space
Across window size classes
Behavior

Many layouts can be established based on the relationship of a list and a detail view.

Key use cases for this layout include parent-child pairings of information like:

Text message + conversation
File browser + open folder
Musical artist + album detail
Settings + category detail
Email inbox + selected email
List 
Detail

Usage

Use the list-detail view for displaying browsable content and quickly showing details.

Examples include: showing a series of conversations and a text message; browsing files and seeing their details; or browsing multiple albums and seeing individual track information in an adjacent view.

Simplified diagram of:

List area
Detail area

Dividing space

The most basic list-detail views for compact, medium, and expanded layouts

A list-detail view uses two panes: one for a list or group of items and the other for a detailed view. Depending on the window class, the two panes may appear together in the same layout or across separate layouts.

List-detail canonical layouts use the same pane guidance as all single and two-pane layouts, including special behavior for foldables.

Window size class (dp)	Visible panes
Compact (0-599)	1 pane
Medium (600-839)	1 (recommended) or 2 panes
Expanded (840+)	2 panes
Large (1200-1599)	2 panes
Extra-large (1600+)	2 panes

Across window size classes

Compact
Use a single-pane layout
Only one view is visible at a time (either list or detail)
Phone in portrait orientation
Closed foldable
Tablet in split-screen mode

Medium
Use a single-pane layout for information-dense content or longer interactions
Foldable open flat
Tablet in portrait orientation

Use a two-pane layout for information-dense content, or quicker interactions
To avoid cramped pane widths, use a bottom navigation bar or modal navigation drawer with two-pane layouts in medium only
Foldable open flat
Tablet in portrait orientation

Expanded, large, and extra-large
Use a two-pane layout

Phone in landscape orientation
Tablet in landscape orientation

Behavior

Single vs two-pane
Back button: Appears in detail view only for single-pane layouts
Selected state: Appears only in list view for two-pane layouts
Visual focus: Use explicit and implicit grouping to direct focus in two-pane layouts

 Navigating between list and detail views is different in each layout

Transitioning between layouts

The amount of available space is dynamic and changes based on user behavior, such as rotating or unfolding a device, or entering a multi-window mode.

A two-pane list-detail layout adapts to a one-pane layout when the device is rotated, changing from expanded to medium window class

No selected list item

The single-pane screen shows the list, and the two-pane screen shows placeholder content in the detail pane.

In some use cases, such as multi-select, the pane last interacted with should remain visible when switching back to single-pane view.

If no item in the list view is selected when a foldable is opened, the revealed pane displays an empty detail view

Selected list item

When going from a single- to two-pane view, both panes should be shown. The selected item’s details are visible.

When going from a two- to single-pane view, the result depends on the product behavior:

Generally, the detail pane should be shown on the single-pane view, and an app bar appears.
However, if the product supports selected list items without navigating deeper, like multi-select, it can show the list view instead with the item selected.
The most important rule is consistency. If the single pane showed the list view before, it should revert to the list view when going back to a single pane.

If an item in the list is selected when a foldable is opened, the revealed pane displays that item’s detail view

If an item in the list is selected when a foldable is closed, the list view is hidden and the detail view is shown in the single pane

If no list item is selected, list pane remains visible and detail pane hides.

In some use cases, such as multi-select, the pane last interacted with should remain visible.

If no item in the list is selected when a foldable is closed, the detail view is hidden and the list view is shown in the single pane

In most cases, a state should be saved when navigating between detail views. Detail views with read and unread content fall into this use case.

The scroll position of a detail view is retained even after navigating to other list items
