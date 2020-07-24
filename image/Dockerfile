#
# thomasweise/docker-texlive-thin
#
# This is an image with a thin TeX Live installation and several
# fonts and tools.
# Source: http://github.com/thomasWeise/docker-texlive-thin/
# License: GNU GENERAL PUBLIC LICENSE, Version 3, 29 June 2007
# The license applies to the way the image is built, while the
# software components inside the image are under the respective
# licenses chosen by their respective copyright holders.
#
FROM ubuntu:20.10
MAINTAINER Thomas Weise <tweise@hfuu.edu.cn>

ENV LANG=C.UTF-8
ENV DEBIAN_FRONTEND=noninteractive

RUN echo "Performing initial clean-up and updates." &&\
    apt-get -y update &&\
    apt-get -y --fix-missing install &&\
    apt-get -y --with-new-pkgs upgrade &&\
# prevent doc and man pages from being installed
# the idea is based on https://askubuntu.com/questions/129566
    echo "Preventing documentation and man-pages from being installed." &&\
    printf 'path-exclude /usr/share/doc/*\npath-include /usr/share/doc/*/copyright\npath-exclude /usr/share/man/*\npath-exclude /usr/share/groff/*\npath-exclude /usr/share/info/*\npath-exclude /usr/share/lintian/*\npath-exclude /usr/share/linda/*\npath-exclude=/usr/share/locale/*' > /etc/dpkg/dpkg.cfg.d/01_nodoc &&\
# remove doc files and man pages already installed
    echo "Removing documentation and man pages already installed." &&\
    rm -rf /usr/share/groff/* /usr/share/info/* &&\
    rm -rf /usr/share/lintian/* /usr/share/linda/* /var/cache/man/* &&\
    rm -rf /usr/share/man &&\
    mkdir -p /usr/share/man &&\
    find /usr/share/doc -depth -type f ! -name copyright -delete &&\
    find /usr/share/doc -type f -name "*.pdf" -delete &&\
    find /usr/share/doc -type f -name "*.gz" -delete &&\
    find /usr/share/doc -type f -name "*.tex" -delete &&\
    find /usr/share/doc -type d -empty -delete || true &&\
    mkdir -p /usr/share/doc &&\
    mkdir -p /usr/share/info &&\
# install utilities
    echo "Installing utilities." &&\
    apt-get install -f -y --no-install-recommends apt-utils || exit 1 &&\
# get and update certificates, to hopefully resolve mscorefonts error
    echo "Getting and updating certificates to help with mscorefonts." &&\
    apt-get install -f -y --no-install-recommends ca-certificates || exit 1  &&\
    update-ca-certificates &&\
# install some utilitites
    echo "Installing more fonts and utilities." &&\
    apt-get install -f -y --no-install-recommends \
          curl \
          fonts-dejavu \
          fonts-dejavu-core \
          fonts-dejavu-extra \
          fontconfig \
          xz-utils || exit 1  &&\
# install the microsoft core fonts
    echo "Installing Microsoft core fonts." &&\
    echo "ttf-mscorefonts-installer msttcorefonts/accepted-mscorefonts-eula select true" | debconf-set-selections &&\
    echo "ttf-mscorefonts-installer msttcorefonts/present-mscorefonts-eula note" | debconf-set-selections &&\
    curl --output "/tmp/ttf-mscorefonts-installer.deb" "http://ftp.de.debian.org/debian/pool/contrib/m/msttcorefonts/ttf-mscorefonts-installer_3.7_all.deb" &&\
    apt install -f -y --no-install-recommends "/tmp/ttf-mscorefonts-installer.deb" || exit 1 &&\
    rm -f "/tmp/ttf-mscorefonts-installer.deb" &&\
# we make sure to contain the EULA in our container
    echo "Adding Microsoft EULA to image." &&\
    curl --output "/root/mscorefonts-eula" "http://corefonts.sourceforge.net/eula.htm" &&\
# install TeX Live and ghostscript as well as other tools
    echo "Installing TeX Live and ghostscript and other tools." &&\
    apt-get install -f -y --no-install-recommends \
          cm-super \
          dvipng \
          ghostscript \
          make \
          latexmk \
          lmodern \
          poppler-utils \
          psutils \
          t1utils \
          tex-gyre \
          texlive-base \
          texlive-binaries \
          texlive-font-utils \
          texlive-fonts-recommended \
          texlive-latex-base \
          texlive-latex-recommended \
          texlive-luatex \
          texlive-pstricks \
          texlive-xetex || exit 1 &&\
# Install other necessary latex packages without
# user tree and then and clean up.
    echo "Initializing user tree." &&\
    tlmgr init-usertree || exit 1 &&\
    echo "tlmgr init-usertree done successfully" &&\
    updmap-sys || exit 1 &&\
    echo "initial updmap-sys done successfully" &&\
# tlmgr will call updmap-sys unsuccessfully for
# reasons I do not understand, so while it seemingly
# succeeds, it will still fail
    echo "Installing packages." &&\
    tlmgr install awesomebox \
                  babel-german \
                  background \
                  filehook \
                  fontawesome5 \
                  footmisc \
                  footnotebackref \
                  fvextra \
                  ly1 \
                  mdframed \
                  mweights \
                  needspace \
                  pagecolor \
                  siunitx \
                  sourcecodepro \
                  sourcesanspro \
                  ucharcat \
                  unicode-math \
                  xecjk || true &&\
    echo "tlmgr install completed, moving fonts" &&\
    cat /root/texmf/web2c/updmap.cfg >> /usr/share/texmf/web2c/updmap.cfg &&\
    rm -rf /root/texmf/tlpkg || true &&\
    rm -rf /root/texmf/web2c || true &&\
    cp -n -r /root/texmf/* /usr/share/texlive/texmf-dist/ || exit 1 &&\
    rm -rf /root/texmf || true &&\
    rm -rf /root/texmf-var || true &&\
    echo "installed packages merged and /root/texmf deleted" &&\
# update latex and font system files:
# Since I am not sure about the proper order,
# we just do this twice.
# iteration 1
    echo "Updating LaTeX and font system files, iteration 1." &&\
    fc-cache -fv || exit 1 &&\
    echo "fc-cache succeeded" &&\
    texhash --verbose || exit 1 &&\
    echo "texhash completed successfully" &&\
    updmap-sys || exit 1 &&\
    echo "updmap-sys completed successfully" &&\
    mktexlsr --verbose || exit 1 &&\
    echo "mktexlsr succeeded" &&\
    fmtutil-sys --quiet --missing &&\
    echo "fmtutil-sys --missing completed successfully" &&\
    fmtutil-sys --quiet --all > /dev/null &&\
    echo "fmtutil-sys --all completed successfully" &&\
# iteration 2
    echo "Updating LaTeX and font system files, iteration 2." &&\
    fc-cache -fv || exit 1 &&\
    echo "fc-cache succeeded" &&\
    texhash --verbose || exit 1 &&\
    echo "texhash completed successfully" &&\
    updmap-sys || exit 1 &&\
    echo "updmap-sys completed successfully" &&\
    mktexlsr --verbose || exit 1 &&\
    echo "mktexlsr succeeded" &&\
    fmtutil-sys --quiet --missing &&\
    echo "fmtutil-sys --missing completed successfully" &&\
    fmtutil-sys --quiet --all > /dev/null &&\
    echo "fmtutil-sys --all completed successfully" &&\
# Hopefully, the installation and merging of latex packages
# into the system-wide latex installation has worked...
# delete texlive sources and other potentially useless stuff
    echo "Removing potentially useless stuff from LaTeX installation." &&\
    rm -rf /usr/share/texmf/source || true &&\
    rm -rf /usr/share/texlive/texmf-dist/source || true &&\
    rm -rf /usr/share/texlive/texmf-dist/doc/ || true &&\
    find /usr/share/texlive -type f -name "readme*.*" -delete &&\
    find /usr/share/texlive -type f -name "README*.*" -delete &&\
    rm -rf /usr/share/texlive/release-texlive.txt || true &&\
    rm -rf /usr/share/texlive/doc.html || true &&\
    rm -rf /usr/share/texlive/index.html || true &&\
    rm -rf /usr/share/texlive/texmf-dist/fonts/source &&\
    rm -rf /usr/share/texlive/texmf-dist/tex/latex/pst-poker &&\
# clean up all temporary files
    echo "Cleaning up temporary files." &&\
    apt-get clean -y &&\
    rm -rf /var/lib/apt/lists/* &&\
    rm -f /etc/ssh/ssh_host_* &&\
# delete man pages and documentation
    echo "Deleting man pages and documentation." &&\
    rm -rf /usr/share/man &&\
    mkdir -p /usr/share/man &&\
    find /usr/share/doc -depth -type f ! -name copyright -delete &&\
    find /usr/share/doc -type f -name "*.pdf" -delete &&\
    find /usr/share/doc -type f -name "*.gz" -delete &&\
    find /usr/share/doc -type f -name "*.tex" -delete &&\
    find /usr/share/doc -type d -empty -delete || true &&\
    mkdir -p /usr/share/doc &&\
    rm -rf /var/cache/apt/archives &&\
    mkdir -p /var/cache/apt/archives &&\
    rm -rf /tmp/* /var/tmp/* &&\
    find /usr/share/ -type f -empty -delete || true &&\
    find /usr/share/ -type d -empty -delete || true &&\
    mkdir -p /usr/share/texmf/source &&\
    mkdir -p /usr/share/texlive/texmf-dist/source &&\
# run a test: are the LaTeX packages really installed?
    echo "Testing whether the LaTeX packages have been installed correctly." &&\
    cd "/tmp/" &&\
    echo '\documentclass{article}%' > test.tex &&\
    echo '\usepackage{awesomebox}%' >> test.tex &&\
    echo '\usepackage{background}%' >> test.tex &&\
    echo '\usepackage{filehook}%' >> test.tex &&\
    echo '\usepackage{fontawesome5}%' >> test.tex &&\
    echo '\usepackage{footmisc}%' >> test.tex &&\
    echo '\usepackage{footnotebackref}%' >> test.tex &&\
    echo '\usepackage{fvextra}%' >> test.tex &&\
    echo '\usepackage[LY1]{fontenc}%' >> test.tex &&\
    echo '\usepackage{mdframed}%' >> test.tex &&\
    echo '\usepackage{mweights}%' >> test.tex &&\
    echo '\usepackage{needspace}%' >> test.tex &&\
    echo '\usepackage{pagecolor}%' >> test.tex &&\
    echo '\usepackage{siunitx}%' >> test.tex &&\
    echo '\usepackage{sourcecodepro}%' >> test.tex &&\
    echo '\usepackage{sourcesanspro}%' >> test.tex &&\
    echo '\begin{document}%' >> test.tex &&\
    echo 'This is a test!' >> test.tex &&\
    echo '\end{document}%' >> test.tex &&\
    echo "Now testing LaTeX." &&\
    latex -halt-on-error -interaction=nonstopmode -no-shell-escape test.tex &&\
    mv test.tex a &&\
    rm -f test.* &&\
    mv a test.tex &&\
    echo "Now testing PdfLaTeX." &&\
    pdflatex -halt-on-error -interaction=nonstopmode -no-shell-escape test.tex &&\
    rm -rf /tmp/* /var/tmp/* &&\
    echo '\documentclass{article}%' > test.tex &&\
    echo '\usepackage{ucharcat}%' >> test.tex &&\
    echo '\usepackage{unicode-math}%' >> test.tex &&\
    echo '\usepackage{xecjk}%' >> test.tex &&\
    echo '\begin{document}%' >> test.tex &&\
    echo 'This is a test!' >> test.tex &&\
    echo '\end{document}%' >> test.tex &&\
    echo "Now testing XeLaTeX." &&\
    xelatex -halt-on-error -interaction=nonstopmode -no-shell-escape test.tex &&\
# final cleanup
    echo "Performing final clean-up." &&\
    rm -rf /tmp/* /var/tmp/* &&\
    echo "Setup completed."

ADD scripts /bin/
