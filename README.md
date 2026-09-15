15 sep 26

Tricorder-0.6.31.apk

say CLOCK for clock

SOL button now includes moon phase tile

improved summarization

<img width="1512" height="2016" alt="IMG_5515" src="https://github.com/user-attachments/assets/871aee67-2cd4-42dd-9b6d-ffc43904a783" />

<img width="1512" height="2016" alt="IMG_5516" src="https://github.com/user-attachments/assets/ae3158a6-e263-47d0-8139-1a395150183a" />

<img width="1512" height="2016" alt="IMG_5519" src="https://github.com/user-attachments/assets/53c83324-67f6-4269-b00a-7ed452f781bf" />

<img width="1512" height="2016" alt="IMG_5518" src="https://github.com/user-attachments/assets/fa5a1901-595b-403c-af8c-8d735dfb5c44" />

<img width="1512" height="2016" alt="IMG_5517" src="https://github.com/user-attachments/assets/893c6d6a-62e6-4898-95fc-9192c7c4918c" />


14 sep 26

how to install the termux backend for tricorder 

https://github.com/jimpames/tricorder/blob/main/TERMUX-BACKEND-INSTALL.md


demo of aircraft detection / airframe photo / airframe fact card / departure and destination city photo and description

https://youtube.com/shorts/yuzfF6pPAzw?si=lsaTtwIQfht102_t

<img width="580" height="884" alt="Screenshot 2026-09-14 123508" src="https://github.com/user-attachments/assets/f536eb72-76da-4711-8771-e1e6858001e0" />

## Sprint 13 Sep 2026 — knowledge banks + overhead cards

use:

Tricorder-0.6.29.apk

Mk I now treats an overhead contact as an object you can inspect, and treats
Memory Alpha / MEDICAL as spoken reference banks rather than another menu.

### Memory Alpha
- Inquiry only. No face button. Say **MEMORY ALPHA** or, in one breath,
  **memory alpha Airbus A320**.
- Simple English Wikipedia titles and extracts, English Wikipedia fallback,
  local article cache.
- Bare wake opens the bank, speaks “online, speak a search,” and arms the mic.
  `TERMINATE` or CLOSE leaves.
- Typed SEARCH field on the sheet if you do not want to talk.

### MEDICAL
- Same voice pattern: **MEDICAL** or **medical pink eye** /
  **medical puncture wound**.
- WikiMed (`mdwiki.org`) is the title catalog only. Its REST article API is
  Cloudflare-blocked from the phone, which is why early builds showed titles
  and nothing else.
- Body text and photographs load from English Wikipedia. Related titles stay
  tappable.
- Field briefing, not a clinician.

### Overhead flight card
Audio Sense + aircraft watch + Earth link + a GPS fix.
- OpenSky box around the last fix, adsbdb type / owner / tail, route when
  published.
- Photograph, in order: adsbdb thumb → Planespotters hex/reg → airport-data
  Mode-S → JetPhotos worker. JPEG is written into the field log.
- **FROM / TO** cities are buttons.
- **Photo** or **AIRFRAME FACTS** opens the type / owner / Mode-S / Wikipedia
  card. The same photo rides on that card.
- Robot voice, metric or English from Settings, departure and destination
  spoken when known.

### City dossiers
Touch FROM or TO.
- Wikipedia extract and thumbnail.
- REST Countries (capital, population, region) when a country can be guessed.
- CIA World Factbook snippet when the country is in the onboard GEC table.
- Spoken.
- **BACK TO AIRFRAME** restores the same craft photo so the other city can be
  opened.

### Pin, do not flash
YAMNet was rewriting the CRT every dwell and wiping the picture. The card is
pinned until **DISMISS AIRFRAME**. Audio still classifies and can still speak
gunfire / siren / vehicle lines. CLOSE on the city or airframe sheet also
restores a pinned card.

### Help / manual
Help sheet, spoken `HELP`, and on-device `manual.html` match Field Operations
Manual Rev 0.6.28. Sideload **0.6.29** for that briefing on the phone.

12 sep 26

tricorder now summarizes findings

use

Tricorder-0.6.24.apk

from files above

source = project-tricorder-0.6.24-src.zip - files above

<img width="3024" height="4032" alt="IMG_5476" src="https://github.com/user-attachments/assets/638c573a-caf8-4c63-8296-18e072ec2f36" />

new manual

https://github.com/jimpames/tricorder/blob/main/Project-Tricorder-MkI-Operations-Manual-0.6.24.pdf


10 sep 26

TRICORDER: live demo on galaxy fold 3:

<img width="482" height="853" alt="tricorder" src="https://github.com/user-attachments/assets/49e9b0af-8972-48f7-bf79-f1676c58daf0" />


https://youtube.com/shorts/JeAU4BzDGUk?si=g0wyWh-ZSoBQceSM

https://youtube.com/shorts/GYyf1QLLB5k?si=Hp_tbPeMQINHa9T5

https://youtube.com/shorts/nV7PTRG0XbM?si=uJBINfflsWdC_8p0


use:
Tricorder-0.6.22.apk

see new manual

- robo voice always option on top row under IPL light - slide ROBO to the right to turn on off
- - ROBO will show ON illuminated GREEN when active

- flight detection now has departure and arrival city
- flight detection has option for metric or English units
- flight detection now says tail ID NXXXX altitude feet / miles per hour departure destination
- flight detection always uses robo voice
- artificial horizon display baro pressure and gps altitude, in addition to rose and g meter and speed

- new command: DIAGNOSTICS in inquiry mode, gives tricorder diags

- new command in DICTATION mode - PUNCTUATION - so you need a semicolon? say PUNCTUATION semicolon; question, period, etc

9 sep 26

now with dictation, AI summarization, mind mapping - and the ability to store a dictated note as a log

- powerful dictation verbal editor

say DICTATION in inquiry mode to activate

- see the new manual

use:
Tricorder-0.6.18.apk from above repo
  

8 sep 26

design deep dive

https://www.linkedin.com/pulse/project-tricorder-device-ai-id-objects-sounds-birdcalls-jim-ames-trxnc/


now detects weapons and violence

use
Tricorder-0.6.14-audio-watch.apk

https://youtube.com/shorts/rLpW3YuSCiU?si=Hmnh-mO4w-bs3vBm


6 sep 26

0.6.8 release is below

video - project tricorder: tech deep dive

https://youtu.be/SsJDWRsaqX0?si=8AzlIHKMKePz-gNA


here is the entire project, all source, and the APK

[click here to download tricorder project zip with 0.6.8 APK and source sha256:f98dd7e897471698d0831d71e960aa1ad9b94dc9d1e105c30e721f1625add4a1
](https://github.com/jimpames/tricorder/releases/download/rent-a-hal-tricorder-0.6.8-release/project-tricorder-0.6.8.zip)


<img width="2752" height="1536" alt="Offline_Field_Instrument_Architecture_Overview" src="https://github.com/user-attachments/assets/16b24f39-88ec-4964-930f-45b1f5bc5482" />




<img width="1376" height="768" alt="tricorder_architecture_schematic" src="https://github.com/user-attachments/assets/3004229e-d1a9-4916-983e-72624366b2db" />
<img width="2752" height="1536" alt="Multimodal_Field_Instrument_Interface" src="https://github.com/user-attachments/assets/23d3b2a0-7e24-4b6a-99f3-46d787250d5f" />

use this file from the files above:
Tricorder-0.6.8-earth-scroll.apk

new features demo:
https://www.youtube.com/shorts/-FUhsDLzfAs

full system in RELEASE linked below, but be sure to use APK above.

- SOL scanner button
- - shows planetary orbit current map, space weather
  -  Earth radio propagation forecast
  -   hyperlocal weather forecast for 72 hours
  

5 sep 26

use this apk:Tricorder-0.6.4-sky.apk

now includes:
- gravity meter and speed on artificial horizon (GPS-based speed, earth speeds only)

- star map
- satellite map

  
use this APK:

Tricorder-0.6.1-js-fix.apk

use the RELEASE linked below - but the APK above...

- notes - this apk polished the tricorder GUI with help , credits and automated floura/fauna/birdnet/translator setup
- as well as noaa and flight and safety alerts

  
<img width="2752" height="1536" alt="Multimodal_Field_Instrument_Project_Overview" src="https://github.com/user-attachments/assets/1ecda407-d98a-486f-a824-b53cbf68b241" />
4 sep 26
video overview

https://youtube.com/shorts/2JjI1MxxGUY?si=5H23hg_kP8r1eEXP


aircraft detection and alerting

<img width="525" height="866" alt="Screenshot 2026-09-04 104602" src="https://github.com/user-attachments/assets/56d44cba-4a3e-4085-979a-5d36bc6296a6" />


use this APK - most current:
Tricorder-0.5.6-horizon-toggle.apk

notice the horizon display can toggle off/on now

the full release is linked below - the APK in the release is stale - use the APK named above

the tricorder now detect and alerts when there are aircraft in your vicinity.

https://youtube.com/shorts/PnPMwuEA5jc?si=Y8mBrhqxKxq0XdQE

https://youtube.com/shorts/z220l6EItAs?si=x6JupEeVmamj7n_S


now includes artificial horizon and e-compass

<img width="3024" height="4032" alt="IMG_5437" src="https://github.com/user-attachments/assets/5a9165b9-89b0-4a68-a39a-b0ca9e1bba3c" />



use the release zip for backend etc

then use the APK above in the repo: 
Tricorder-0.5.4-horizon-watch.apk


WARNING!

** THE TRICORDER ARTIFICIAL HORIZON / E-COMPASS  IS NOT FAA CERTIFIED OR SPACE RATED.

** 

** THE TRICORDER MUST NEVER BE USED AS A PRIMARY NAVIGATION INSTRUMENT FOR ANY FLYING CRAFT OR SPACESHIP



3 sep 26

use the release, below - get the zip - 

then use this apk from the above files:

Tricorder-0.5.3-viewport.apk

- now supports 50 language automatic spoken translation

<img width="527" height="873" alt="Screenshot 2026-09-04 044435" src="https://github.com/user-attachments/assets/aabdd68a-1f2b-4a1e-aae7-a13a5e89d9a2" />


translator demo video

https://youtube.com/shorts/MDNXRhQnc2M?si=viq78HgO-b_f4A2w

 
1 sep 26

flora ID with video - class M planet

https://youtube.com/shorts/yCOJXfKc20M?si=Ehe8zYvTRzmgWLbV

people are asking if this is a toy mockup or is it a real tricorder?

of course it's real!! if you want a toy there are many - go find one

- why would I post a mock up!?

- it's 100% real and functional - read the manual - for this tricorder

- obviously this is my project rent-a-hal but for android - and delivered as an appliance.

<img width="2752" height="1536" alt="Project_Software-Defined_Field_Instrument" src="https://github.com/user-attachments/assets/2a6c991e-e88c-41c9-8b55-6b70998a1311" />

you can get the entire up to date project from the release now, 
or use the zip in the repo and then the recommended latest apk

- this release has tricorder-to-tricorder interlink, see the user guide.
  https://github.com/jimpames/tricorder/blob/main/Project-Tricorder-MkI-Operations-Manual.pdf
  

https://github.com/jimpames/tricorder/releases/download/rent-a-hal-AI-for-android-v-0-9-beta/project-tricorder-0.4.1.zip****

<img width="1512" height="2016" alt="IMG_5325" src="https://github.com/user-attachments/assets/52a294b8-0483-47bf-a845-8fcf753b4407" />
 1 sep 26
 
use the file

Tricorder-0.4.1-inquiry-fix.apk

this version identifies floura, fauna, sounds, birdcalls. 

as well as combines chat, vision, imagine and speech on device.

features a log you can replay.


after you install all the zip stuff


<img width="361" height="604" alt="Screenshot 2026-08-31 073349" src="https://github.com/user-attachments/assets/0aab7583-c9f7-462b-82cd-3da4feaec14e" />

now w on device audio classification - use apk in repo

https://youtube.com/shorts/gERrC3hePBA?feature=share


now w birdcall identification
https://youtube.com/shorts/qsihwQrngNE?si=FOVGWK_o1j5Hkm61



<img width="522" height="872" alt="Screenshot 2026-09-01 130105" src="https://github.com/user-attachments/assets/f9e28264-7dd7-4d4f-834b-e8f0d919583c" />

# tricorder

live demo video
https://youtube.com/shorts/OVZqemHC8wk?si=uQSmCym5YTsL-y6H

talking machine vision demo
https://youtube.com/shorts/wSPW4QW086M?si=nUyQKdyEGCV8ra9c

rent-a-hal tricorder:  chat, vision, imagine and speech for android 100 % on device AI with multi-modal UX
<img width="2752" height="1536" alt="Offline_AI_on_Legacy_Hardware" src="https://github.com/user-attachments/assets/7c3a4934-5358-436d-ac8a-45ba219d345f" />

overview:

https://youtu.be/SjvG9m4q6wI?si=_dH8VCtRIlI9Ycd8


<img width="1512" height="2016" alt="IMG_5297" src="https://github.com/user-attachments/assets/ac3d6c19-08e6-46ca-b958-0a81080350bd" />


<img width="1512" height="2016" alt="IMG_5296" src="https://github.com/user-attachments/assets/242f0cf6-5986-41dd-b49d-436b714a6309" />


<img width="3024" height="4032" alt="IMG_5293" src="https://github.com/user-attachments/assets/f7c516e8-a0e6-40d0-a22d-19592ace5b0d" />


<img width="1010" height="2184" alt="IMG_5291" src="https://github.com/user-attachments/assets/b06b436c-81e7-4a08-be48-e064315d77e7" />


<img width="1512" height="2016" alt="IMG_5278" src="https://github.com/user-attachments/assets/f8216517-bd74-4e5e-bb7a-c15d9dc3c1d8" />
