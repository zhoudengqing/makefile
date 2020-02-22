srcs := $(wildcard *.cc)
objs := $(patsubst %.cc, %.o, $(srcs))
sub_dir := hello world

.PHONY : $(target) $(sub_dir)
$(target) : $(sub_dir) $(objs)
	$(LD) $(LDFLAGS) $(sub_dir:=/$(target)) $(objs) -r -o $@

$(sub_dir):
	$(MAKE) -C $@
%.o : %.cc
	$(CXX) $(CXXFLAGS) $< -c
%.d : %.cc
	$(CXX) $(CXXFLAGS) $< -MM > $@.$$$$; \
	sed 's/$*.o: /$*.d $*.o: /' < $@.$$$$ > $@; \
	$(RM) $@.$$$$
sinclude $(srcs:%.cc=%.d)
.PHONY : clean
clean :
	$(RM) $(target) $(objs) $(srcs:%.cc=%.d)
	for dir in $(sub_dir);do \
		$(MAKE) -C $$dir clean; \
	done

