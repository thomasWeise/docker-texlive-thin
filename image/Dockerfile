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
FROM ubuntu:21.10
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
    (find /usr/share/doc -type d -empty -delete || true) &&\
    mkdir -p /usr/share/doc &&\
    mkdir -p /usr/share/info &&\
# install utilities
    echo "Installing utilities." &&\
    apt-get install -f -y --no-install-recommends apt-utils &&\
# get and update certificates, to hopefully resolve mscorefonts error
    echo "Getting and updating certificates to help with mscorefonts." &&\
    apt-get install -f -y --no-install-recommends ca-certificates &&\
    update-ca-certificates &&\
# install some utilitites
    echo "Installing more fonts and utilities." &&\
    apt-get install -f -y --no-install-recommends \
          curl \
          fonts-dejavu \
          fonts-dejavu-core \
          fonts-dejavu-extra \
          fonts-noto-cjk \
          fontconfig \
          xz-utils &&\
# install the microsoft core fonts
    echo "Installing Microsoft core fonts." &&\
    echo "ttf-mscorefonts-installer msttcorefonts/accepted-mscorefonts-eula select true" | debconf-set-selections &&\
    echo "ttf-mscorefonts-installer msttcorefonts/present-mscorefonts-eula note" | debconf-set-selections &&\
    curl --output "/tmp/ttf-mscorefonts-installer.deb" "http://ftp.de.debian.org/debian/pool/contrib/m/msttcorefonts/ttf-mscorefonts-installer_3.7_all.deb" &&\
    apt install -f -y --no-install-recommends "/tmp/ttf-mscorefonts-installer.deb" &&\
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
          latex-cjk-chinese \
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
          texlive-lang-chinese \
          texlive-latex-base \
          texlive-latex-recommended \
          texlive-luatex \
          texlive-pstricks \
          texlive-xetex &&\
# Install other necessary latex packages without
# user tree and then and clean up.
    echo "Initializing user tree." &&\
    tlmgr init-usertree &&\
    echo "tlmgr init-usertree done successfully" &&\
    echo "Using TeXLive 2020 repository." &&\
# see https://tex.stackexchange.com/questions/540429
    tlmgr repository add ftp://tug.org/historic/systems/texlive/2020/tlnet-final &&\
    tlmgr repository list &&\
    tlmgr repository remove http://mirror.ctan.org/systems/texlive/tlnet &&\
    tlmgr option repository ftp://tug.org/historic/systems/texlive/2020/tlnet-final &&\
# updating via tlmgr
    echo "Updating TeXLive via tlmgr." &&\
    tlmgr --verify-repo=none update --self &&\
    tlmgr --verify-repo=none update --all &&\
    echo "Invoking updmap-sys." &&\
# calling updmap-sys
    updmap-sys &&\
    echo "initial updmap-sys done successfully" &&\
# tlmgr will call updmap-sys unsuccessfully for
# reasons I do not understand, so while it seemingly
# succeeds, it will still fail
    echo "Installing packages." &&\
    (tlmgr --verify-repo=none install awesomebox \
                                      babel-german \
                                      background \
                                      fandol \
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
                                      unicode-math || true) &&\
    echo "tlmgr install completed, moving fonts" &&\
    cat /root/texmf/web2c/updmap.cfg >> /usr/share/texmf/web2c/updmap.cfg &&\
    (rm -rf /root/texmf/tlpkg || true) &&\
    (rm -rf /root/texmf/web2c || true) &&\
    cp -n -r /root/texmf/* /usr/share/texlive/texmf-dist/ &&\
    (rm -rf /root/texmf || true) &&\
    (rm -rf /root/texmf-var || true) &&\
    echo "installed packages merged and /root/texmf deleted" &&\
    echo "now deleting useless packages" &&\
    cd /usr/share/texlive/texmf-dist/tex/latex/ &&\
    rm -rf a0poster a4wide achemso acro* actuarial* bewerbung biochemistr* \
           bithesis bizcard bondgraph* bookshelf bubblesort carbohydrates \
           catechis cclicenses changelog cheatsheet circui* commedit comment \
           contracard course* csv* currvita* cv* dateiliste* denisbdoc \
           diabetes* dinbrief directory dirtytalk duck* duotenzor \
           dynkin-diagrams easy* elegant* elzcards emoji enigma es* \
           etaremune europasscv europecv exam* exceltex exercis* exesheet \
           ffslides fibeamer fink fithesis fixme* fjodor fla* flip* form* \
           fonttable forest g-brief gauss gcard gender genealogy* git* \
           gloss* gmdoc* HA-prosper hackthefootline halloweenmath hand* \
           harnon-cv harpoon harveyballs he-she hobby hpsdiss ifmslide \
           image-gallery invoice* interactiveworkbook isorot isotope \
           istgame iwhdp jknapltx jlabels jslectureplanner jumplines \
           kalendarium kantlipsum keystroke kix knitting* knowledge \
           komacv* labels* ladder lectures lettr* lewis logbox magaz \
           mail* makebarcode mandi mathexam mceinleger mcexam \
           medstarbeamer menu* mi-solns minorrevision minutes mla-paper \
           mnotes moderncv modernposter moderntimeline modiagram moodle \
           multiaudience mwe my* neuralnetwork newspaper nomen* papermas \
           pas-* pb-diagram permutepetiteannonce phf* philex \
           phonenumbers photo piff* pinlabel pixelart plantslabels \
           pmboxdraw pmgraph polynom* powerdot ppr-prv practicalreports \
           pressrelease probsoln productbox progress* proofread protocol \
           psbao psfrag* pst-* python qcircuit qcm qrcode qs* quantikz \
           quicktype quiz2socrative quotchap qyxf-book ran* rcs* \
           readablecv readarray realboxes recipe* rectopma \
           reflectgraphics register reotex repeatindex rterface \
           rulercompass runcode sa-tikz sauerj schedule schemabloc \
           schooldocs scratch* scsnowman sdrt semant* seminar sem* \
           sesstime setdeck sf298 sffms shadethm shdoc shipunov \
           signchart simple* skb skdoc skeldoc skills skrapport \
           smartdiagram spectralsequences sslides studenthandouts svn* \
           swfigure swimgraf syntaxdi syntrace synttree table-fct \
           tableaux tabu talk tasks tdclock technics ted texmate \
           texpower texshade threadcol ticket ticollege todo* \
           topiclongtable tqft tucv tufte-latex twoup uebungsblatt uml \
           unravel upmethodology uwmslide vdmlisting venndiagram \
           verbasef verifiche versonotes vhistory vocaltract was \
           webquiz williams willowtreebook worksheet xbmks xcookybooky \
           xcpdftips xdoc xebaposter xtuthesis xwatermark xytree ya* \
           ycbook ydoc yplan zebra-goodies zed-csp zhlipsum ziffer zw* &&\
    cd /tmp/ &&\
# update latex and font system files:
# Since I am not sure about the proper order,
# we just do this twice.
# iteration 1
    echo "Updating LaTeX and font system files, iteration 1." &&\
    fc-cache -fv &&\
    echo "fc-cache succeeded" &&\
    texhash --verbose &&\
    echo "texhash completed successfully" &&\
    updmap-sys &&\
    echo "updmap-sys completed successfully" &&\
    mktexlsr --verbose &&\
    echo "mktexlsr succeeded" &&\
    fmtutil-sys --quiet --missing &&\
    echo "fmtutil-sys --missing completed successfully" &&\
    fmtutil-sys --quiet --all > /dev/null &&\
    echo "fmtutil-sys --all completed successfully" &&\
# iteration 2
    echo "Updating LaTeX and font system files, iteration 2." &&\
    fc-cache -fv &&\
    echo "fc-cache succeeded" &&\
    texhash --verbose &&\
    echo "texhash completed successfully" &&\
    updmap-sys &&\
    echo "updmap-sys completed successfully" &&\
    mktexlsr --verbose &&\
    echo "mktexlsr succeeded" &&\
    fmtutil-sys --quiet --missing &&\
    echo "fmtutil-sys --missing completed successfully" &&\
    fmtutil-sys --quiet --all > /dev/null &&\
    echo "fmtutil-sys --all completed successfully" &&\
# Hopefully, the installation and merging of latex packages
# into the system-wide latex installation has worked...
# delete texlive sources and other potentially useless stuff
    echo "Removing potentially useless stuff from LaTeX installation." &&\
    (rm -rf /usr/share/texmf/source || true) &&\
    (rm -rf /usr/share/texlive/texmf-dist/source || true) &&\
    (rm -rf /usr/share/texlive/texmf-dist/doc/ || true) &&\
    find /usr/share/texlive -type f -name "readme*.*" -delete &&\
    find /usr/share/texlive -type f -name "README*.*" -delete &&\
    (rm -rf /usr/share/texlive/release-texlive.txt || true) &&\
    (rm -rf /usr/share/texlive/doc.html || true) &&\
    (rm -rf /usr/share/texlive/index.html || true) &&\
    rm -rf /usr/share/texlive/texmf-dist/fonts/source &&\
    rm -rf /usr/share/texlive/texmf-dist/tex/latex/pst-poker &&\
    rm -rf /usr/share/texlive/README &&\
    rm -rf /usr/share/texlive/texmf-dist/texdoctk &&\
    rm -rf /usr/share/texlive/texmf-dist/texdoc &&\
    echo "Removing doc and other useless things." &&\
    rm -rf /usr/share/doc &&\
    mkdir -p /usr/share/doc &&\
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
    (find /usr/share/doc -type d -empty -delete || true) &&\
    mkdir -p /usr/share/doc &&\
    rm -rf /usr/share/texlive/texmf-dist/doc &&\
    rm -rf /var/cache/apt/archives &&\
    mkdir -p /var/cache/apt/archives &&\
    rm -rf /tmp/* /var/tmp/* &&\
    (find /usr/share/ -type f -empty -delete || true) &&\
    (find /usr/share/ -type d -empty -delete || true) &&\
    mkdir -p /usr/share/texmf/source &&\
    mkdir -p /usr/share/texlive/texmf-dist/source &&\
# run a test: are the LaTeX packages really installed?
    echo "Testing whether the LaTeX packages have been installed correctly." &&\
    cd "/tmp/" &&\
    echo '\\documentclass{article}%' > test.tex &&\
    echo '\\usepackage{awesomebox}%' >> test.tex &&\
    echo '\\usepackage{background}%' >> test.tex &&\
    echo '\\usepackage{filehook}%' >> test.tex &&\
    echo '\\usepackage{fontawesome5}%' >> test.tex &&\
    echo '\\usepackage{footmisc}%' >> test.tex &&\
    echo '\\usepackage{footnotebackref}%' >> test.tex &&\
    echo '\\usepackage{fvextra}%' >> test.tex &&\
    echo '\\usepackage[LY1]{fontenc}%' >> test.tex &&\
    echo '\\usepackage{mdframed}%' >> test.tex &&\
    echo '\\usepackage{mweights}%' >> test.tex &&\
    echo '\\usepackage{needspace}%' >> test.tex &&\
    echo '\\usepackage{pagecolor}%' >> test.tex &&\
    echo '\\usepackage{siunitx}%' >> test.tex &&\
    echo '\\usepackage{sourcecodepro}%' >> test.tex &&\
    echo '\\usepackage{sourcesanspro}%' >> test.tex &&\
    echo '\\begin{document}%' >> test.tex &&\
    echo 'This is a test!' >> test.tex &&\
    echo '\\end{document}%' >> test.tex &&\
    echo "Now testing LaTeX." &&\
    latex -halt-on-error -interaction=nonstopmode -no-shell-escape test.tex &&\
    mv test.tex a &&\
    rm -f test.* &&\
    mv a test.tex &&\
    echo "Now testing PdfLaTeX." &&\
    pdflatex -halt-on-error -interaction=nonstopmode -no-shell-escape test.tex &&\
    rm -rf /tmp/* /var/tmp/* &&\
    echo '\\documentclass{article}%' > test.tex &&\
    echo '\\usepackage{xeCJK}%' >> test.tex &&\
    echo '\\usepackage{ucharcat}%' >> test.tex &&\
    echo '\\usepackage{unicode-math}%' >> test.tex &&\
    echo '\\begin{document}%' >> test.tex &&\
    echo 'This is a test!' >> test.tex &&\
    echo '\\end{document}%' >> test.tex &&\
    echo "Now testing XeLaTeX." &&\
    xelatex -halt-on-error -interaction=nonstopmode -no-shell-escape test.tex &&\
# final cleanup
    echo "Performing final clean-up." &&\
    rm -rf /tmp/* /var/tmp/* &&\
    echo "Setup completed."
