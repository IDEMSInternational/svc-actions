# The image the PDF builds run in: the runner's own image with LaTeX, Chromium's
# system libraries and Java already installed, so a build no longer spends 8-12
# minutes installing them. build.yml runs its job in it when the PDF is built
# (input `pdf-image`); a translation repository's book job names it in its own
# `container:` (see translation-pdf). Both check /etc/svc-pdf-image and skip
# their install step when it is there.
#
# Built and pushed to ghcr.io by .github/workflows/pdf-image.yml when this file
# changes on main, or by hand. Nothing else rebuilds it, so the TeX Live in it
# changes only when someone decides it should.
#
# The base is what the runner's `ubuntu-latest` label runs, so node (for the
# JavaScript actions), sudo and the rest of a normal job are all still here.
FROM catthehacker/ubuntu:act-latest

# The packages build.yml and translation-pdf installed on every run, plus the
# French that translation-pdf adds and the Java that Okapi Tikal needs.
RUN apt-get update \
 && DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends \
      texlive-xetex texlive-latex-extra texlive-fonts-recommended \
      texlive-fonts-extra texlive-science texlive-lang-french \
      fonts-dejavu fonts-noto-core latexmk fontconfig \
      openjdk-17-jre-headless \
 && rm -rf /var/lib/apt/lists/*

# The print build screenshots interactive figures with the Chromium that
# pretext downloads itself; this is the system libraries it needs.
RUN npx --yes playwright@1 install-deps chromium \
 && rm -rf /var/lib/apt/lists/* /root/.npm

RUN printf 'svc-pdf image, built %s\n' "$(date -u +%Y-%m-%d)" >/etc/svc-pdf-image
