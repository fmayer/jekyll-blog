---
title: "A Paper a Week #3: SoundWave: Using the Doppler Effect to Sense Gestures"
layout: post
date: 2015-03-15 22:00:00 BT
categories: apaw
---

I almost feel like I cheated by picking such a short paper – it has been a busy week, though. As the paper is only 4 pages long, this post won’t be particularly long, either.

The paper presents a way of sensing gestures with two pieces of hardware found in almost every laptop that has recently been produced: a microphone and a loadspeaker. The loudspeaker generates a tone in the inaudible range that can be produced by off-the shelve loudspeakers: around 18 – 22 kHz. An initial calibration step determines the exact frequency, as some may be impossible to produce on a given setup. This is then recorded by the microphone, the signal is Fourier-transformed using the FFT. After some data sanitising based on e.g. maximum observable gesture speed (the wording is quite funny: “Informal tests with multiple people indicated that the fastest speed at which they could move their hands in front of a laptop was about 3.9 m/sec”), movement is detected by a significant frequency shift, and velocity, direction and proximity to the device are extracted. I think this is a valuable presentation of the usefulness of the FFT in real-world software applications.

Using the Doppler-effect to detect movement certainly is not a new idea, but the paper presented that it is viable to use hardware generally available rather than specifically built higher-frequency equipment.

The paper can be obtained here: [PDF] [1].

[1]: http://research.microsoft.com/en-us/um/redmond/groups/cue/publications/guptasoundwavechi2012.pdf "Paper"
