all: read_shell_scripts.html
clean:
	rm -f read_shell_scripts.html
read_shell_scripts.html: read_shell_scripts.rst
	rst2html --halt=1 $< $@

