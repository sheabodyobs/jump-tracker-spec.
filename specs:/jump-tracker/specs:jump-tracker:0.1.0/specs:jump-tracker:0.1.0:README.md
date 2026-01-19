{\rtf1\ansi\ansicpg1252\cocoartf2706
\cocoatextscaling0\cocoaplatform0{\fonttbl\f0\froman\fcharset0 Times-Roman;}
{\colortbl;\red255\green255\blue255;\red0\green0\blue0;}
{\*\expandedcolortbl;;\cssrgb\c0\c0\c0;}
\margl1440\margr1440\vieww28600\viewh17440\viewkind0
\deftab720
\pard\tx28571\pardeftab720\partightenfactor0

\f0\fs24 \cf0 \expnd0\expndtw0\kerning0
\outl0\strokewidth0 \strokec2 # Jump Tracker Spec \'97 v0.1.0\
\
Spec-first definitions for extracting jump events and metrics from single-camera video.\
\
This repository contains specs only.\
No implementation code lives here.\
\
## What this spec produces\
- Event timeline: contact start/end, takeoff, landing\
- Metrics: ground contact time (GCT), flight time, counts\
- Optional: coarse foot-contact pattern, coarse joint angles (if available)\
\
## Scope (v0.1.0)\
- Single camera (side/front/rear allowed; view is declared)\
- Offline processing allowed (no real-time requirement)\
- One athlete in frame (primary subject)\
- Outputs are deterministic given the same input video\
\
## Non-goals (v0.1.0)\
- Multi-camera fusion\
- Real-time latency guarantees\
- Medical or injury-risk claims\
- Personalized programming recommendations\
\
## How to change this spec\
- Patch bump (0.1.x): clarifications, non-breaking additions, more tests\
- Minor bump (0.x.0): output schema changes, semantic changes to events/metrics\
}