all: makepkg.html
clean:
	rm -f makepkg.html
makepkg.html: makepkg.rst
	rst2html --halt=1 $< $@
