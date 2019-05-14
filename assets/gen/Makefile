objects = attr_list.dot.png attr_list.dot.svg logo_hires.png

all : $(objects)

attr_list.dot.png : attr_list.dot
	dot -Tpng -O attr_list.dot

attr_list.dot.svg : attr_list.dot
	dot -Tsvg -O attr_list.dot

logo_hires.png : logo.svg
	inkscape -z logo.svg -e logo_hires.png

.PHONY : clean
clean :
	-rm $(objects)
