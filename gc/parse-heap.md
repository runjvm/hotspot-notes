ensure_parsibility() is always necessary, although the tlab should have been filled after it's created. I am not sure why, but if you don't ensure_parsibility() again, the rest of the tlab will be null area, instead of fillers.

