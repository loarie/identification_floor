# Identification Floor Demo

This is a lightweight sandbox app for exploring how **iNaturalist identifications** affect an observation’s **community taxon**, **observation taxon**, and **quality grade** as additional IDs are added.

The app pulls in a real iNaturalist observation and lets you **simulate** different identification sequences. Nothing is written back to iNaturalist — this is strictly a read-only demo. [Try it here](https://scottloarie.com/identification_floor/)

## What this is for

- Demonstrating how IDs roll up (or don’t) to a Community Taxon  
- Visualizing how quality grade changes as IDs are added  
- Helping people understand proposed or existing ID logic through hands-on experimentation  
- Supporting discussion and volunteer testing without impacting real observations

## What it does (today)

- Loads an observation from iNaturalist  
- Simulates adding IDs and shows how:
  - Community Taxon  
  - Observation Taxon  
  - Quality Grade  
  would change on iNaturalist

## What it does *not* do

- Does **not** write anything back to iNaturalist  
- Does **not** represent official iNaturalist behavior in all edge cases

## Known gaps / not yet implemented

- “Based on the Evidence, can the Community Taxon be improved?” (DQA)  
- Community Taxon opt-out behavior  
- Some edge cases around subspecies and disagreement

## Status

This is an experimental prototype built quickly to explore ideas and facilitate discussion. Behavior should be treated as illustrative, not authoritative.
