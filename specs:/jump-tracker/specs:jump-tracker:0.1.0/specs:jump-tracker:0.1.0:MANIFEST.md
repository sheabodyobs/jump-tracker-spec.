{\rtf1\ansi\ansicpg1252\cocoartf2706
\cocoatextscaling0\cocoaplatform0{\fonttbl\f0\froman\fcharset0 Times-Roman;}
{\colortbl;\red255\green255\blue255;\red0\green0\blue0;}
{\*\expandedcolortbl;;\cssrgb\c0\c0\c0;}
\margl1440\margr1440\vieww28600\viewh17440\viewkind0
\deftab720
\pard\pardeftab720\partightenfactor0

\f0\fs24 \cf0 \expnd0\expndtw0\kerning0
\outl0\strokewidth0 \strokec2 spec: jump-tracker\
version: 0.1.0\
status: draft\
\
files:\
  - README.md\
  - MANIFEST.md\
  - specs/jump-tracker.spec.md\
\
interfaces:\
  input:\
    - video_uri\
    - fps\
    - width\
    - height\
    - camera_view (optional)\
  output:\
    - events[]\
    - metrics\{\}\
    - confidence\{\}\
\
acceptance_criteria:\
  - Defines input fields and required/optional constraints\
  - Defines event types and ordering rules\
  - Defines GCT computation and ambiguity handling\
  - Defines confidence + human-readable notes\
  - Includes spec-level acceptance tests\
}