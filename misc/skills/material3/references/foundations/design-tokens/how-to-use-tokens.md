---
source_url: https://m3.material.io/foundations/design-tokens/how-to-use-tokens
fetched_at: 2026-04-29
section: foundations
---

Design tokens
Design tokens are the building blocks of all UI elements. The same tokens are used in designs, tools, and code.
Overview
How to use tokens
On this page
Design tokens
Resources
What’s a design token?
Why are tokens important?
Deciding if tokens are right for you
Tokens & Material Design
Parts of a token name
Classes of tokens
Contexts
On this page
Design tokens
Download Material baseline tokens
Use tokens in Figma

Download Material baseline tokens

Material Design’s baseline theme includes design tokens and default values. Download the theme as a Design System Package (DSP) to customize, collaborate on, and use in your own designs and product code.  Learn about the DSP JSON format

Use tokens in Figma

To begin, install the Material Theme Builder Figma plugin from the community page.

Generate tokens
Open Figma and navigate to: Plugins > Material Theme Builder > Open Plugin
Select Get started, this will create material-theme with baseline values by default. Color and text styles will begin populating the right hand design panel. When your tokens are fully generated, your artboard will contain tonal palettes for light and dark color schemes, as well as a default type scale.
Your tokens are now represented as Figma styles that can be used throughout your designs

Update token values
Using the Material Theme Builder Figma plugin (updates colors only)
Open Figma and navigate to: Plugins > Material Theme Builder > Open Plugin
Choose the colors. Updated color and text styles will begin populating the right hand design panel.
Your updated tokens are now represented as Figma styles that can be used throughout your designs

Using Figma styles
In Figma, navigate to the file in which the tokenized style is defined. Shortcut: right click the style in the right hand sidebar and select Go to style definition.
In the right hand sidebar, hover over the style you want to update and select the adjust icon when it appears. Or, right click the style in the style picker and select Edit style.
Make your changes to the token name, description, properties, etc. using the Edit style panel. Close the panel when finished.

Use tokens in product mock-ups
Instead of manually setting the color or typography for elements in a layout, apply the Figma styles representing your design tokens. This helps ensure that developers will correctly understand and apply your design choices.

Use tokens with the Material Design Kit
Duplicate the Material Design Kit in Figma
Navigate to Plugins > Material Theme Builder > Open plugin
With components selected, select swap. This will swap the baseline Material token style values with your own generated token style values.

Export tokens
Open Figma and navigate to: Plugins > Material Theme Builder > Open Plugin
Navigate to the Export tab. Select the format you want to export for (Android, Jetpack Compose)
Name your .zip file and select Save

Your tokens are ready to share!
