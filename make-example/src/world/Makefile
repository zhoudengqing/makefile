srcs := $(wildcard *.cc)
objs := $(patsubst %.cc, %.o, $(srcs))

$(target) : $(objs)
	$(LD) $(LDFLAGS) $^ -r -o $@

%.o : %.cc
	$(CXX) $(CXXFLAGS) $< -c 
%.d : %.cc
	$(CXX) $(CXXFLAGS) $< -MM > $@.$$$$; \
	sed 's/$*.o: /$*.d $*.o: /' < $@.$$$$ > $@; \
	$(RM) $@.$$$$
sinclude $(srcs:%.cc=%.d)
.PHONY : clean
clean :
	$(RM) $(objs) $(srcs:%.cc=%.d) $(target)
