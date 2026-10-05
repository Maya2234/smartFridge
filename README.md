###project write up
# Smart Fridge Interface
This project is meant to simulate a UI for any refrigerator for the purpose of aiding the user in having access to internet features in the kitchen. Functionality includes timers, temprature controls, grocery list, and decorative settings.

## Design

### Affordances and physical properties
The UI clearly shows temprature, timer, and grocery list controls afforded to the user. The location is fixed and very large and heavy.

### 2.2 Assumed smart features

These are the capabilities I assumed the fridge has:

senses door open closed
shows inside temprature
displays time

### 2.3 User needs (from interviews)
What do you find yourself needing in the kitchen that is typically difficult or inconvient to actually preform?
Could you benefit from an additional internent capable device in your kitchen?
how do you currently set timers and view the time in the kitchen? Does that work well for you?

I learned that often times users need to set timers and check the time in the kitchen when often is in an unknown location or inacessible. Additionally, many manually timed devices in the kitchen (oven, microwave) often become out of sync with the actual time due to lack of internet access. Users often recognize items needed to be added to the grocery list when in the kitchen and unable to actually preform the action. When phone is in hand, the need has been forgotten.

### 2.4 User needs and design requirements

Easy way to add, remove and view shopping list as well as timers. Options to customize display for aestetic purposes. 

### 2.5 Sketches



### 2.6 Feedback on the sketch



### Page layout
Program is split into variables, functions, and html elements. There is a seperate sylesheet. Timer, keypad, and input have their own files because I have sourced that code externally from an internet resource. Runes have been used to automatically reload page when necessary.

### Basic controls and indicators (Level 1)

The UI shows current time, and 3 basic functions: temprature adjustment, timer functionality, and grocery list functionality. These are draggable to customize and maximize information displayed. 

The settings menu offers options for background color, background text, and text color. These choices can then be saved as a template. There is a feature that offers users the ability to schedule templates within a 24 hour period. Users can then simulate the 24 hour period in a 2 minute window to view the template schedule they have programmed.

## 5. Future Work

-Audio (music, timer audio, custom template switch audio)
-send grocery list to iphone, or make collaborative with mobile
--sticky note feature

---

## 6. AI Documentation

I used claude and copilot for assistance in explaining concepts in svelte and troubleshooting performance issues.

```
