#指定根目录，子目录，头文件目录和目标文件目录变量
export cur_dir := $(shell pwd)
sub_dir := main src
export inc_dir := $(cur_dir)/include
bin_dir := $(cur_dir)/bin
#定义中间生成的目标文件
export target := built-in.o
#定义编译器相关的变量
export CROSS_COMPILE :=
export CXX := $(CROSS_COMPILE)g++
export AR := $(CROSS_COMPILE)ar
export LD := $(CROSS_COMPILE)ld
export CXXFLAGS += -I$(inc_dir) -g -Wall
export LDFLAGS += 
#定义最终生成文件
bin_target := hello_world

#定义生成规则
.PHONY : all $(bin_target) $(sub_dir)
all : $(bin_target)
$(bin_target) : $(sub_dir)
	$(CXX) $(CXXFLAGS) $(LDFLAGS) $(^:=/$(target)) -o $@; \
	mv $(bin_target) $(bin_dir)

$(sub_dir):
	$(MAKE) -C $@

#定义清除规则
.PHONY : clean
clean :
	$(RM) $(bin_dir)/$(bin_target)
	for dir in $(sub_dir);do \
		$(MAKE) -C $$dir clean; \
	done

