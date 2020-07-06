# Open Reactor Simulator

This is a open-source web-based real-time nuclear reactor simulator for
education, demonstration, and fun. Users are presented a control panel with
instruments and control as well as a live schematic diagram of the plant
indicating basic status of the core, pumps, heat exchangers, coolant flow, and
so on. 

# Requirements

## R1: The dashboard shall react to user actions within 1 second of actuation

To provide interactive education, the simulator must respond in near real time
to user control actions.  Speed is more important than physical accuracy. One
server should support at least 20 concurrent users.

## R2: The system shall capture the attention of a 14-year-old for 4 minutes.

It's gotta be fun, interesting, and exciting to achieve its outreach mission.

## R3: The system shall simulate an AP1000-like LWR, a MSR, and a SFR. 

The system shall be designed in such a way that a variety of reactors can be
simulated. The first use case can be a particular reactor, but design decisions
should keep in mind the desirability of various reactor types (e.g. LWR, MSR,
HTGR, SFR, ...). This will make the system relevant to the widest number of
users and developers.

## R4: The system shall not require any specialized software to be installed for use

To best educate the largest number of people, the system must be easy to
access on common household PCs without the burden of acquiring and installing
specialized software.  Thus, standard technologies must be used in the
presentation of the UI.

## R5: The system shall be operable on a mobile device, a normal PC with a 15" monitor, and a PC with a 50" monitor.

For broad flexibility in private and public, it should work on a wide variety of screen sizes

## R6: The system shall be widely available for development by interested parties, worldwide.

To ensure continued development and upgrades, the source code of the
system must be readable and editable broadly. 

# Technology stack (proposal/straw man)

## Django backend. 
Handle sessions, API, various reactor configurations, and (maybe) physics. If
Django/Python is too slow to be performant, use something else for physics,
including (possibly) client-side physics (if that's even possible, would it have
to be in JS?). This could use [PyRK for kinetics](http://pyrk.github.io/), for
example. 

This will need asynchronous session-based data to allow a number of people to
run instances of the simulator at the same time. 

## HTML5/Javascript/AJAX front-end
A web-based front-end will easily satisfy the "easy installation" requirement.
This can take advantage of libraries like
[ChartJS](https://github.com/nagix/chartjs-plugin-streaming),
[Gauge.coffee](http://bernii.github.io/gauge.js/), [Material
UI](https://material-ui.com/), etc.

## Open-source on GitHub
This will satisfy the "easy development" requirement. Probably a MIT license, but maybe GPL

## Hosted on a whatisnuclear VPS
We have servers and sysadmins that can get us started. If it gets popular we
should have a load-balancing strategy or try shifting physics to the client.

# Key Decisions

## Finalize UI technology stack

**Decision:**
We will use the [SVG.js](https://svgjs.com/docs/2.7/) library to display the animated 
reactor scene. 

**Rationale:**
We need to bring life to various reactor components on the screen, and
Javascript-controlled SVG animations are a good way to do this. This library is
simple, easy to work with, well-documented, and passes initial performance
checks. 

In a proof-of-concept, @partofthething built a simple pump animation combining
JS-generated SVG (pump rotor) as well as Inkscape-drawn SVG (pump casing) and
was able to change the rotor animation speed with a HTML5 range input. This was
confirmed to work in Firefox and Chromium on Linux and Android. Here is the 
code as a [fiddle](https://jsfiddle.net/partofthething/pw18q0hm/8/). 

To stress test I got 100 rotors spinning at different speeds and they all rendered
just fine on a 2013 laptop. I also got a little primary/secondary loop thing going
which you can see [here](https://partofthething.com/spintest-svgjs.html)

Alternatives Considered:

* Velocity.js --- Was not able to figure out how to change animation speed interactively. 
* Vanilla js --- Code quickly got too big for simple animations
* GSAP --- License seemed slightly odd

Concerns:
* None! This library looks great.

For gauges and graphs and indicators and whatnot, we should not feel constrained to SVG.js,
and other libraries like

* [ChartJS](https://github.com/nagix/chartjs-plugin-streaming),
* [Gauge.coffee](http://bernii.github.io/gauge.js/), 
* [Material UI](https://material-ui.com/)

should all be leveraged as appropriate. 


## Finalize physics technology stack

**Decision:**
[Django](https://www.djangoproject.com/) will be used on the server dealing
with physics, reactor details, and session.  

**Rationale:**
Many simple nuclear reactor simulator projects (e.g.
[PyRK](http://pyrk.github.io/)) are easily accessible from Python, and many
potential collaborators are familiar with the Python language. Django is the
only web framework the team has experience with, and it doesn't seem to have
any show-stopping characteristics with respect to the requirements. 

We successfully built a simple proof-of-concept using
[Channels](https://channels.readthedocs.io/en/latest/) to get low-latency
full-duplex interactions between the client and the server. Interactions
between a UI element, a trivial calculation on the server, and a UI response
were seemingly instantaneous. The *scope* concept in that library should allow
multiple independent simulations to run simultaneously. 

Concerns:

* Django is not the fastest web framework out there, and performance may become an issue, 
  especially with many users.

Alternatives considered:

* Honestly, none.


## Pick a 1-D transient coupled neutronics/thermal hydraulics simulator
**Decision:**

(Still in investigation). It looks like building out own 1-D transient simulator is going
to be the best bet here. I think re-building [Sandia's 1984 code,
TOPAZ](https://www.osti.gov/servlets/purl/5437631) in Python, looks really
doable, and passing its results to the front end via Django Channels.

**Rationale:**
TOPAZ is 1-D and can handle 1- or 2-phase flow, compressible or incompressible, etc.
It's relatively straightforward and well documented. It even has a series of test problems
we can validate it against. If this ran on awesome computers in 1984, it should run fine
for simple problems in the web browser today. 

I have started coding it up in Python. We will use steam tables at first, but since it
uses Pressure and Internal energy to look things up, any coolant can be added in later. 

Concerns:

* Is it too crazy to write our own transient code for this? Possibly, but it's also pretty
  neat and fun. Let's see how it goes. 


## Choose license
**Decision:**

**Rationale:**

## Choose first plant to model
Probably best to just do a PWR.

**Decision:**

**Rationale:**

# Project Plan

## Phase 1: Technology Stack Shakedown
Build fundamental dummy components of the interface and try them out. Start with
the proposed tech stack and just set up the most trivial case possible, with a
UI slider passing data to a back-end and seeing the slider value squared (or
something) show up on a live plot. Do some experiments to see how much load such
a system might have on a server and how responsive it is with a full control
panel/dashboard. Try out a few simple animations (change color of a square based
on slider value) to shake down the components of the reactor animation. 

**Dummy kinetics solver**

The applet at https://live.whatisnuclear.com uses Django channels to solve the actual PKE
live and plots with javascript. Source for the physics solver is
[here](https://github.com/partofthething/keepinpace). Source for webapp coming soon.

## Phase 2: Design and implement key systems

### Backend
Design and implement a DB schema that allows a variety of reactors to be
simulated. 

Design and implement supersimple physics engines (0-D, 1-D T/H, etc.). 

### Frontend
Design and implement the control panel and reactor animation

## Phase 3: Build and test first reactor simulator
Build out a full-on PWR simulator. Test and adjust.

## Phase 4: Build additional simulations
Other reactors, anyone?


# Related Projects

## IAEA Nuclear Reactor Simulators for education and training
The IAEA
[simulators](https://www.iaea.org/topics/nuclear-power-reactors/nuclear-reactor-simulators-for-education-and-training)
project is very related. Many of these don't meet the requirements of easy
access/use/development or multi-reactor capabilities.

## nuclearpowersimulator.com
[This web game](http://nuclearpowersimulator.com/) is very similar in concept
and operation to what we want. We just want it to be a little bit more
interactive, with better graphics, more live plots, and possibly sound effects.

