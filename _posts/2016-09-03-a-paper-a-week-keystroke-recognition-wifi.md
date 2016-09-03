---
title: "A Paper a Whatever #15: Keystroke Recognition Using WiFi Signals"
layout: post
date: 2016-09-03 16:15:00 CET
categories: apaw
---
This paper is about detecting a user's keypresses using the interference the movements of the fingers cause in WiFi signals. This sounds scary and like something out of a cheesy spy movie; nevertheless it is not quite time to get out your tinfoil hat, as (at least as demonstrated) this is only accurate in controlled conditions where equipment is specifically set-up and the user is instructed not to move their head. It does not seem like there is anything that makes it inherently impossible to do in uncontrolled conditions.

The introduction talks about various other interesting approaches other people have taken to recognize keystrokes, including matching on the distinctive sound different keypresses produce or their electromagnetic signature.  There is also prior work in using radio interference patterns, but the authors of this paper took the different approach of only using over-the-counter equipment for doing so. I was not aware at all that there was this much research going on on essentially remote key-logging.

Their approach, however, exploits something known as CSI values – sadly, no, that does not stand for Crime Scene Investigation. CSI stands for Channel State Information and is a set of values that WiFi devices use to optimize their transmission among multiple channels. 

They observed that it is possible to find signatures for different movements of the fingers in those CSI values, but they faced the problem that they needed to determine when a key-press started and ended, and noise was also a problem. They would denoise the signal by first applying low-pass filtering and then then applying Principal Component Analysis on the data retrieved from the CSI of various channels – they noticed that while the changes were not the same in all of them, they were strongly correlated. So they applied PCA, discarding the highest variance component.

Mean Absolute Deviations are then used to find parts of the timeseries that are high in variance and correspond to keypresses by comparing them to empirically obtained thresholds. The thus found timeseries are then used to train a k Nearest Neighbour model, which would classify each unknown sample with the class that the majority of its k nearest neighbours from the training data-set correspond to. They use Dynamic Time Warping as a distance metric, which tries to align two data-sets by non-linearly warping the time axis until they match. This allows them to compare the signatures of keypresses, even if they are pressed longer or shorter than in the training data.

This model is then trained with 30 data-samples per user per key, and numbers for accuracy are given. This is with the caveat that the experiment was done under controlled conditions, in particular the users were asked to type one key at a time and not to move their heads or other body parts. Training the model independently for each user, they managed to achieve 93.5 % keypress recognition accuracy in continuously typed sentences.

Find the paper [here] [1].

[1]: https://www.sigmobile.org/mobicom/2015/papers/p90-aliA.pdf "Paper as PDF"
