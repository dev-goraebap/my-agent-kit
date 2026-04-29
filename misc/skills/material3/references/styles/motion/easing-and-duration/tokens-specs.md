---
source_url: https://m3.material.io/styles/motion/easing-and-duration/tokens-specs
fetched_at: 2026-04-29
section: styles
---

Easing and duration
Easing and duration create responsive and expressive motion
Applying easing and duration
Tokens & specs
On this page
Easing and duration
Suggested easing and duration pairs
Easing
Duration
On this page
Easing and duration
Tokens
Easing
Duration

star
Note:In the expressive update, components and motion now use the motion physics system, which uses springs. Products should migrate to the new system. The easing and duration system is still used for transitions and can be used by teams that haven't yet updated to GM3 Expressive, but is no longer maintained.

Tokens

Motion easing and duration can be implemented using easing and duration tokens. Learn more about design tokens

Motion
arrow_drop_down
search
view_list
expand_all
Standard, Android
arrow_drop_down
grid_3x3

Custom composite with 2 properties

grid_3x3

Custom composite with 2 properties

grid_3x3

Custom composite with 2 properties

grid_3x3

Custom composite with 2 properties

grid_3x3

Custom composite with 2 properties

grid_3x3

Custom composite with 2 properties

folder
Spring
keyboard_arrow_down
folder
Easing
keyboard_arrow_down
folder
Duration
keyboard_arrow_down
folder
Style
keyboard_arrow_down

Easing

Emphasized easing set

This set is the most common because it captures the expressive style of M3.

Emphasized

Emphasized decelerate

Emphasized accelerate

Info/Platform	Emphasized	Emphasized decelerate
	Emphasized accelerate

Token	md.sys.motion.easing.emphasized	md.sys.motion.easing.emphasized.decelerate	md.sys.motion.easing.emphasized.accelerate
Android	pathInterpolator(M 0,0 C 0.05, 0, 0.133333, 0.06, 0.166666, 0.4 C 0.208333, 0.82, 0.25, 1, 1, 1)	PathInterpolator(0.05f, 0.7f, 0.1f, 1f)	PathInterpolator(0.3f, 0f, 0.8f, 0.15f)
CSS	N/A (Use Standard as a fallback)	cubic-bezier(0.05, 0.7, 0.1, 1.0)	cubic-bezier(0.3, 0.0, 0.8, 0.15)
Flutter	easeInOutCubicEmphasized	Cubic(0.05, 0.7, 0.1, 1.0);	Cubic(0.3, 0.0, 0.8, 0.15);
iOS	N/A (Use Standard as a fallback)	ControlPoints:0.05f:0.7f:0.1f:1.0f];	ControlPoints:0.3f:0.0f:0.8f:0.15f];
After Effects	Use After Effects Easing Panel (download)

Standard easing set

This set is used for simple, small, or utility-focused transitions.

Standard

Standard decelerate

Standard accelerate

	Standard	Standard decelerate	Standard accelerate
Token	md.sys.motion.easing.standard	md.sys.motion.easing.standard.decelerate	md.sys.motion.easing.standard.accelerate
Android	PathInterpolator(0.2f, 0f, 0f, 1f)	PathInterpolator(0f, 0f, 0f, 1f)	PathInterpolator(0.3f, 0f, 1f, 1f)
CSS	cubic-bezier(0.2, 0.0, 0, 1.0);	cubic-bezier(0, 0, 0, 1);	cubic-bezier(0.3, 0, 1, 1);
Flutter	Cubic(0.2, 0.0, 0, 1.0);	Cubic(0, 0, 0, 1);	Cubic(0.3, 0, 1, 1);
iOS	ControlPoints:0.2f:0.0f:0.0f:1.0f	ControlPoints:0.0f:0.0f:0.0f:1.0f	ControlPoints:0.3f:0.0f:1.0f:1.0f];
After Effects	Use After Effects Easing Panel (download)

Duration

Short durations

These are used for small utility-focused transitions.

Token	Value
md.sys.motion.duration.short1	50ms
md.sys.motion.duration.short2	100ms
md.sys.motion.duration.short3	150ms
md.sys.motion.duration.short4	200ms

Selection controls have a short duration of 200ms with Standard easing

Medium durations

These are used for transitions that traverse a medium area of the screen.

Token	Value
md.sys.motion.duration.medium1	250ms
md.sys.motion.duration.medium2	300ms
md.sys.motion.duration.medium3	350ms
md.sys.motion.duration.medium4	400ms

A FAB expanding into a Sheet uses a 400ms duration with Emphasized easing

Long durations

These durations are often paired with Emphasized easing. They're used for large expressive transitions.

Token	Value
md.sys.motion.duration.long1	450ms
md.sys.motion.duration.long2	500ms
md.sys.motion.duration.long3	550ms
md.sys.motion.duration.long4	600ms

A Card expanding to full screen uses a long 500ms duration with Emphasized easing

Extra long durations

Though rare, some transitions use durations above 600ms. These are usually used for ambient transitions that don't involve user input.

Token	Value
md.sys.motion.duration.extra-long1	700ms
md.sys.motion.duration.extra-long2	800ms
md.sys.motion.duration.extra-long3	900ms
md.sys.motion.duration.extra-long4	1000ms

An ambient carousel auto-advance transition uses an extra long 1000ms duration with emphasized easing
