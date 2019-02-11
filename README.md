# NPRMs

```

wget https://www.cms.gov/Center/Special-Topic/Interoperability/CMS-9115-P.pdf
wget https://www.healthit.gov/sites/default/files/nprm/ONCCuresActNPRM.pdf

pdftohtml  ONCCuresActNPRM.pdf ONC.html
pdftohtml CMS-9115-P.pdf CMS.html
sed -i 's/&#160;/ /g' *.html

pandoc -t gfm  --toc   CMSs.html -o CMSs.md
pandoc -t gfm  --toc   ONCs.html -o ONCs.md

```

View online at:

* http://joshuamandel.com/interop-2019-nprms/CMSs.html#outline
* http://joshuamandel.com/interop-2019-nprms/ONCs.html#outline
