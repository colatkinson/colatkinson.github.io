objects = attr_list.dot.png attr_list.dot.svg

all : $(objects)

attr_list.dot.png : attr_list.dot
	dot -Tpng -O attr_list.dot

attr_list.dot.svg : attr_list.dot
	dot -Tsvg -O attr_list.dot

.PHONY : clean
clean :
	-rm $(objects)
