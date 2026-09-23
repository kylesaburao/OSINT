---
name: image-locator
description: Geolocate a photograph from visible clues, provenance, maps, and optional geometry or timing checks when GPS is missing or unreliable.
---

# Locate a photograph

Find the most specific **camera area** supported by the evidence. The photographed place or landmark may be elsewhere. Start from pixels even when the file supplies a plausible location.

1. **Preserve and inspect.** Keep the supplied original unchanged and record where it came from; make a working copy before any transformation. Note dimensions, orientation, file history, and any metadata claims without treating them as ground truth. If a tag claims a location, camera or lens tags could calibrate geometry, tags conflict, credentials appear, or decoding is suspect, read [metadata and provenance](references/metadata-provenance.md). A metadata-free image continues through the same workflow.
2. **Extract pixel clues.** Record observations separately from place interpretations. Use road details, structures, landscape and text to form competing regions, then sketch the visible layout and search for combinations of mapped features. Read [visual clues and image history](references/visual-clues.md) for regional cues, map-search tactics, OCR and reverse-image provenance. Submit the image or a crop to an external reverse-image service only with the user's authorization; text and public-page searches remain available without an upload.
3. **Test camera-site candidates.** Reconstruct plausible viewpoints from independent maps or original sources; compare road alignment, landmark order, occlusion and terrain, then seek the strongest contradiction. Read [maps and camera geometry](references/map-geometry.md) when perspective, road direction, bearings, field of view, terrain, lens effects or a fitted pose could matter. A landmark address identifies the subject, not the camera.
4. **Apply conditional checks.** If Sun, shadow, Moon, stars, or horizon could constrain direction or time, read [astronomy and timing](references/astronomy-timing.md), including when capture time is unknown. If the scene may have changed, or a credible time window makes weather, tide, a distinctive vessel, or aircraft useful, read [dated imagery and public records](references/dated-imagery-signals.md). Keep uncertain inputs as ranges or alternative hypotheses.
5. **Evaluate and report.** Use [evidence evaluation and reporting](references/evidence-reporting.md) to log each candidate's observations, sources and dates, supporting and conflicting geometry, and confidence. Report a point only if the camera is isolated. Otherwise give a bounded building, block, or area, an approximate representative pin with its extent, and viable alternatives. Label camera and subject coordinates separately and cite the evidence that supports the chosen precision.

Finish when further narrowing needs evidence the case does not supply.
