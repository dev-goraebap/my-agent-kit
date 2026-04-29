---
source_url: https://m3.material.io/components/app-bars/overview
fetched_at: 2026-04-29
section: components
---

# Top app bar

App bars
App bars are placed at the top of the screen to help people navigate through a product
Resources
flutter
android
+3
info
Overview
style
Specs
design_services
Guidelines
head_mounted_device
XR
accessibility_new
Accessibility
On this page
App bars
Availability & resources
M3 Expressive update
Differences from M2

Focus on describing the current page and provide 1–2 essential actions

Displays labels and page navigation controls at the top of the page. (Use a toolbar to display page actions)

Four variants: Search app bar, small, medium flexible, large flexible

On scroll, apply a fill color to separate from body content

Can animate on and off screen with another bar of controls, like a row of chips

Search app bar
Small
Medium flexible
Large flexible

Availability & resources

Type	Resource	Status
Design

Design Kit (Figma)
	
Available

Implementation

Flutter
	
Available

android
Jetpack Compose
	
Available

android
Jetpack Compose: Expressive
	
Available

android
MDC-Android
	
Available

android
MDC-Android: Expressive
	
Available

language
Web
	
Unavailable

language
Web: Expressive
	
Unavailable

M3 Expressive update

May 2025
The new search app bar supports icons inside and outside the search bar, and centered text. It opens the search view component when selected.

The new medium flexible and large flexible app bars come with significant improvements, and should replace medium and large app bars, which are no longer recommended. The small app bar is updated with the same flexible improvements. 

More on M3 Expressive

Variants and naming:

Renamed component from top app bar to app bar

Added search app bar

Medium and large app bars are no longer recommended

Added medium flexible and large flexible app bars with:

Reduced overall height

Larger title text

Subtitle

Left- and center-aligned text options

Text wrapping

More flexible elements for imagery and filled buttons

Added features to small app bar:

Subtitle

Center-aligned text option

More flexible elements for imagery and filled buttons

Search app bar
Small
Medium flexible
Large flexible

Differences from M2

Color: New color mappings and compatibility with dynamic color
On scroll: No drop shadow, instead a color fill creates separation from content
Typography: Larger default text
Layout: Smaller default height

M2: Elevation and a drop shadow raise the top app bar when content is present underneath

M3: On scroll, a color fill overlay separates the app bar from the content beneath
