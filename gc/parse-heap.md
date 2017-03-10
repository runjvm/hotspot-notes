`ensure_parsibility()` is always necessary before parsing a MutablSpace, although the tlabs should have been filled after it's created. I am not sure why, but if you don't ensure_parsibility() again, the unused space of a tlab will be null area, instead of fillers.

With `ensure_parsibility()` on, parsing from bottom to top has no problem, no matter UseNUMA is on or not.
