---
source_url: https://m3.material.io/styles/motion/overview/how-it-works
fetched_at: 2026-04-29
section: styles
---

Motion physics system
The motion physics system makes a UI expressive and easy to use
How it works
Specs
On this page
Motion physics system
A motion system designed for expression
Availability & resources
The basics: Motion schemes
How it works: Springs
Spring tokens
Application
Advanced customizations

A motion system designed for expression

May 2025

Material introduced the motion physics system with M3 Expressive. This new physics-based system makes interactions and transitions feel more alive, fluid, and natural. It represents a new motion language for Google products, and is easier to implement and customize than ever before. 

The physics system is replacing the previous system based on easing and duration.

More on M3 Expressive

Availability & resources
Type	Link	Status
Implementation
	MDC-Android
	Available. Not yet added to components. See specs
Flutter	Unavailable
Jetpack Compose
	Available
Web	Compatible with Compose springs. See specs

The basics: Motion schemes

The physics system has two preset motion schemes: expressive and standard. The motion scheme you choose defines how your product feels. While most motion in a product should use the same scheme, products can make advanced customizations to swap the scheme to emphasize key moments.

Expressive is Material’s opinionated motion scheme, and should be used for most situations, particularly hero moments and key interactions.

The expressive motion scheme overshoots the final values to add bounce

Standard feels more functional with minimal bounce, and should be used for utilitarian products.

The standard motion scheme eases into the final values

Need something other than the preset schemes? Create your own! The physics system makes it easy to create custom motion schemes beyond expressive and standard, while still leveraging theming. Schemes can be easily switched between expressive, standard, or custom as needed.

How it works: Springs

Motion schemes use springs. A spring is a combination of three attributes which control all motion behavior: stiffness, damping, and initial velocity.

Springs are versatile. One spring can apply to many situations, such as transitions, button effects, or gestures. This makes the motion and expression feel consistent throughout the product.

Springs feel natural. Springs are designed to be predictable, like how objects move and bounce. They handle gestures, interruptions, and retargeting animations seamlessly.

All component motion is driven by two tokens: expressive fast spatial and expressive fast effects

Spring tokens

On Jetpack Compose and MDC-Android, these springs are available as spring tokens. Use tokens to easily apply motion to any element, making all motion feel predictable and consistent across multiple platforms. See specs for how to convert springs to other platforms like Web.

There are tokens for spatial movement and effects, with three durations each: default, fast, and slow.

For example, to apply fast, spatial, expressive motion, call the "expressive" motion scheme, then use the token: md.sys.motion.spring.fast.spatial.

Notice that the "expressive" scheme isn't part of the token itself. Rather, it's called at the product level and applied to all tokens. This makes it easier to swap schemes without changing assigned tokens.

Each scheme (expressive, standard) has three speeds (fast, default, slow) for two types of movement (spatial, effects)

Style

Spatial spring tokens are used for animations that move something on screen, for example the x and y position, rotation, size, rounded corners. This spring overshoots the final value and bounces into place.

Spatial springs apply to movement

Spatial springs apply to rotation

Effects spring tokens are used to animate properties such as color and opacity animations, where there shouldn’t be any overshoot. 

Effects springs applied to opacity

Effects springs applied to color

Speed

Spatial and effect spring tokens come in three speeds: default, fast, and slow. Most motion should use the default speed, while smaller elements may use fast and larger elements may use slow.

Speed
	Spatial example
	Effects example

Default
	Animations that partially cover the screen, such as bottom sheet and  expanded navigation rail	Opacity of the content within a  navigation rail
Fast
	Small components, such as switches and  buttons	Color change of the switch handle
Slow
	Full-screen animations	Full-screen content refresh

Spatial motion in fast, default, and slow speeds

Effects motion in fast, default, and slow speeds

Spring tokens work across devices. For example, the spatial fast token will always be faster than default or slow, but the exact values of each token differ depending on if the device is a wearable, phone, or tablet. This ensures the movement feels fast in the context of the device.

Application

Components

On Jetpack Compose, 21 Material components use the motion physics system by default. MDC-Android support is coming soon. To add the motion physics system to other components, including custom-built components, use spring tokens. View full specs

Material components use the physics motion system to feel more expressive

Advanced customizations

There are a few different levels for applying motion. Choose the level that applies best to your product or specific component.

Level 1: Use a default motion scheme

The expressive and standard schemes should be sufficient for all motion needs. On Jetpack Compose, components use these schemes by default.

Switch using the expressive motion scheme

Switch using the standard motion scheme

Level 2: Create a custom motion scheme

On Jetpack Compose, to change the default motion scheme that all components and transitions use, create a custom MotionScheme object, and return different AnimationSpec for each property of the motion scheme.

FAB menu with an extra stiff custom scheme

FAB menu with very low stiffness custom scheme

Level 3: Swap the default motion scheme per element

Why use just one scheme when you can use multiple? On Jetpack Compose, to use one scheme for most of the product, such as expressive, but on certain elements swap it for another scheme, like standard, override the CompositionLocal for that particular composable, screen, or element.
