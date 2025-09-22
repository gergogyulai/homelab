# Plex Codec Compatibility Guide

Plex, when running on various devices, can be quite particular about the different video and audio codecs, and even specific video-audio codec pairs. This guide aims to help mitigate these compatibility issues.

## Understanding Codec Compatibility

It has been observed that Plex often uses client players that are outdated or poorly optimized, leading to unnecessary transcoding even when the underlying device itself is perfectly capable of direct playing the media. This can be a significant source of frustration, as it places an avoidable load on the Plex Media Server.

### Key Factors Affecting Compatibility:

*   **Client Device Capabilities:** Different devices (Smart TVs, streaming sticks, game consoles, mobile devices, web browsers) have varying hardware and software support for codecs.
*   **Codec Profiles and Levels:** Beyond the basic codec (e.g., H.264), specific profiles (e.g., High, Main) and levels within those codecs can also impact compatibility.
*   **Audio Codecs:** Just like video, audio codecs (e.g., AAC, AC3, DTS, TrueHD) also need to be compatible.

## Master Compatibility List

For a comprehensive and up-to-date reference on what various Plex client devices support, please refer to the master list:

*   **Plex Codec Compatibility Master List:** [https://docs.google.com/spreadsheets/u/1/d/15Wf_jy5WqOPShczFKQB28cCetBgAGcnA0mNOG-ePwDc/htmlview](https://docs.google.com/spreadsheets/u/1/d/15Wf_jy5WqOPShczFKQB28cCetBgAGcnA0mNOG-ePwDc/htmlview)
*   **Archived Version (as of 9/22/2025):** [https://web.archive.org/web/20250922063459/https://docs.google.com/spreadsheets/u/1/d/15Wf_jy5WqOPShczFKQB28cCetBgAGcnA0mNOG-ePwDc/htmlview](https://web.archive.org/web/20250922063459/https://docs.google.com/spreadsheets/u/1/d/15Wf_jy5WqOPShczFKQB28cCetBgAGcnA0mNOG-ePwDc/htmlview)

This master list provides detailed information on which video and audio codecs are supported by different Plex client applications and devices, indicating whether direct play is possible or if transcoding will occur.
