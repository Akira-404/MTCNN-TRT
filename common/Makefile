.SUFFIXES:
TARGET?=$(shell uname -m)
ifeq ($(CUDA_INSTALL_DIR),)
$(warning CUDA_INSTALL_DIR variable is not specified, using /usr/local/cuda by default, use CUDA_INSTALL_DIR=<cuda_directory> to change.)
endif
ifeq ($(CUDNN_INSTALL_DIR),)
$(warning CUDNN_INSTALL_DIR variable is not specified, using $(CUDA_INSTALL_DIR) by default, use CUDNN_INSTALL_DIR=<cudnn_directory> to change.)
endif
CUDA_INSTALL_DIR?=/usr/local/cuda
CUDNN_INSTALL_DIR?=$(CUDA_INSTALL_DIR)
CUDA_LIBDIR=lib
CUDNN_LIBDIR=lib64
ifeq ($(TARGET), aarch64)
ifeq ($(shell uname -m), aarch64)
CUDA_LIBDIR=lib64
CC = g++
else
CC = aarch64-linux-gnu-g++
endif
CUCC =$(CUDA_INSTALL_DIR)/bin/nvcc -m64 -ccbin $(CC)
else ifeq ($(TARGET), x86_64)
CUDA_LIBDIR=lib64
CC = g++
CUCC =$(CUDA_INSTALL_DIR)/bin/nvcc -m64
else ifeq ($(TARGET), qnx)
CC = ${QNX_HOST}/usr/bin/aarch64-unknown-nto-qnx7.0.0-g++
CUCC = $(CUDA_INSTALL_DIR)/bin/nvcc -m64 -ccbin $(CC)
else ifeq ($(TARGET), android64)
ifeq ($(NDK_ROOT),)
$(error NDK_ROOT must be set to build for android platforms)
endif
ifeq ($(ANDROID_CC),)
$(error ANDROID_CC must be set to the g++ compiler to build for android 64bit, for example $(NDK_ROOT)/toolschains/aarch64-linux-android-4.9/prebuilt/linux-x86_64/bin/aarch64-linux-android-g++)
endif
ifeq ($(NDK_SYS_ROOT),)
$(error NDK_SYS_ROOT must be set to compiler for android 64bit, for example $(NDK_ROOT)/platforms/android-24/arch-arm64)
endif
CUDA_LIBDIR=lib64
ANDROID_FLAGS=--sysroot=${NDK_SYS_ROOT} -DANDROID -D_GLIBCXX_USE_C99=1 -Wno-sign-compare -D__aarch64__ -Wno-strict-aliasing -Werror -pie -fPIE
COMMON_FLAGS+=$(ANDROID_FLAGS)
COMMON_LD_FLAGS+=$(ANDROID_FLAGS)
CC=$(ANDROID_CC)
CUCC = $(CUDA_INSTALL_DIR)/bin/nvcc -m64 -ccbin $(CC) --compiler-options="--sysroot=${NDK_SYS_ROOT} -DANDROID -D_GLIBCXX_USE_C99=1 -Wno-sign-compare"
TGT_INCLUDES=-I$(NDK_ROOT)/platforms/android-24/arch-aarch64/usr/include -I$(NDK_ROOT)/sources/cxx-stl/gnu-libstdc++/4.9/include -I$(NDK_ROOT)/sources/cxx-stl/gnu-libstdc++/4.9/libs/arm64-v8a/include
TGT_LIBS=-L$(NDK_ROOT)/sources/cxx-stl/gnu-libstdc++/4.9/libs/arm64-v8a
ANDROID=1
else ########
$(error Auto-detection of platform failed. Please specify one of the following arguments to make: TARGET=[aarch64|x86_64|qnx])
endif

ifdef VERBOSE
AT=
else
AT=@
endif

AR = ar cr
ECHO = @echo

SHELL=/bin/sh

#ROOT_PATH=../..
#OUT_PATH=$(ROOT_PATH)/bin
OUT_PATH=.
OUTDIR=$(OUT_PATH)

define concat
$1$2$3$4$5$6$7$8
endef

#$(call make-depend,source-file,object-file,depend-file)
define make-depend
  $(AT)$(CC) -MM -MF $3 -MP -MT $2 $(COMMON_FLAGS) $1
endef

#########################

# These are the directories where I installed TensorRT on my x86_64 PC.
TENSORRT_INCS=-I"/home/ubuntu/TensorRT-8.2.3.0/include"
TENSORRT_LIBS=-L"/home/ubuntu/TensorRT-8.2.3.0/lib"

INCPATHS=-I"$(CUDA_INSTALL_DIR)/include" $(TENSORRT_INCS) -I"/usr/local/include" -I"$(CUDNN_INSTALL_DIR)/include" $(TGT_INCLUDES) -I"../common"
LIBPATHS=-L"$(CUDA_INSTALL_DIR)/$(CUDA_LIBDIR)" $(TENSORRT_LIBS) -L"/usr/local/lib" -L"$(CUDA_INSTALL_DIR)/$(CUDA_LIBDIR)" -L"$(CUDNN_INSTALL_DIR)/$(CUDNN_LIBDIR)" $(TGT_LIBS)

.SUFFIXES:
vpath %.h $(EXTRA_DIRECTORIES)
vpath %.cpp $(EXTRA_DIRECTORIES)

COMMON_FLAGS += -Wall -std=c++11 $(INCPATHS)
ifneq ($(ANDROID),1)
COMMON_FLAGS += -D_REENTRANT
endif
COMMON_LD_FLAGS += $(LIBPATHS) -L$(OUTDIR)

OBJDIR    =$(call concat,$(OUTDIR),/chobj)
DOBJDIR   =$(call concat,$(OUTDIR),/dchobj)

ifeq ($(ANDROID),1)
COMMON_LIBS = -lcudnn -lcublas -lnvToolsExt -lcudart
else
COMMON_LIBS = -lcudnn -lcublas -lcudart_static -lnvToolsExt -lcudart
endif
ifneq ($(TARGET), qnx)
ifneq ($(ANDROID),1)
COMMON_LIBS += -lrt -ldl -lpthread
endif
endif
ifeq ($(ANDROID),1)
COMMON_LIBS += -lculibos -lgnustl_shared -llog
endif

LIBS  =-lnvinfer -lnvparsers -lnvinfer_plugin $(COMMON_LIBS)
DLIBS =-lnvinfer -lnvparsers -lnvinfer_plugin $(COMMON_LIBS)
OBJS   =$(patsubst %.cpp, $(OBJDIR)/%.o, $(wildcard *.cpp $(addsuffix /*.cpp, $(EXTRA_DIRECTORIES))))
DOBJS  =$(patsubst %.cpp, $(DOBJDIR)/%.o, $(wildcard *.cpp $(addsuffix  /*.cpp, $(EXTRA_DIRECTORIES))))
CUOBJS =$(patsubst %.cu, $(OBJDIR)/%.o, $(wildcard *.cu $(addsuffix  /*.cu, $(EXTRA_DIRECTORIES))))
CUDOBJS =$(patsubst %.cu, $(DOBJDIR)/%.o, $(wildcard *.cu $(addsuffix  /*.cu, $(EXTRA_DIRECTORIES))))

CFLAGS=$(COMMON_FLAGS)
CFLAGSD=$(COMMON_FLAGS) -g
LFLAGS=$(COMMON_LD_FLAGS)
LFLAGSD=$(COMMON_LD_FLAGS)

#all: debug release

release : $(OUTDIR)/$(OUTNAME_RELEASE)

debug   : $(OUTDIR)/$(OUTNAME_DEBUG)

test: test_debug test_release

test_debug:
	$(AT)cd $(OUTDIR) && ./$(OUTNAME_DEBUG)

test_release:
	$(AT)cd $(OUTDIR) && ./$(OUTNAME_RELEASE)

ifdef MAC
$(OUTDIR)/$(OUTNAME_RELEASE) : $(OBJS) $(CUOBJS)
	$(ECHO) Linking: $@
	$(AT)$(CC) -o $@ $^ $(LFLAGS) $(LIBS)
	$(foreach EXTRA_FILE,$(EXTRA_FILES), cp -f $(EXTRA_FILE) $(OUTDIR)/$(EXTRA_FILE); )

$(OUTDIR)/$(OUTNAME_DEBUG) : $(DOBJS) $(CUDOBJS)
	$(ECHO) Linking: $@
	$(AT)$(CC) -o $@ $^ $(LFLAGSD) $(DLIBS)
else
$(OUTDIR)/$(OUTNAME_RELEASE) : $(OBJS) $(CUOBJS)
	$(ECHO) Linking: $@
	$(AT)$(CC) -o $@ $^ $(LFLAGS) -Wl,--start-group $(LIBS) -Wl,--end-group
	$(foreach EXTRA_FILE,$(EXTRA_FILES), cp -f $(EXTRA_FILE) $(OUTDIR)/$(EXTRA_FILE); )

$(OUTDIR)/$(OUTNAME_DEBUG) : $(DOBJS) $(CUDOBJS)
	$(ECHO) Linking: $@
	$(AT)$(CC) -o $@ $^ $(LFLAGSD) -Wl,--start-group $(DLIBS) -Wl,--end-group
endif

$(OBJDIR)/%.o: %.cpp
	$(AT)if [ ! -d $(OBJDIR) ]; then mkdir -p $(OBJDIR); fi
	$(foreach XDIR,$(EXTRA_DIRECTORIES), if [ ! -d $(OBJDIR)/$(XDIR) ]; then mkdir -p $(OBJDIR)/$(XDIR); fi;) :
	$(call make-depend,$<,$@,$(subst .o,.d,$@))
	$(ECHO) Compiling: $<
	$(AT)$(CC) $(CFLAGS) -c -o $@ $<

$(DOBJDIR)/%.o: %.cpp
	$(AT)if [ ! -d $(DOBJDIR) ]; then mkdir -p $(DOBJDIR); fi
	$(foreach XDIR,$(EXTRA_DIRECTORIES), if [ ! -d $(OBJDIR)/$(XDIR) ]; then mkdir -p $(DOBJDIR)/$(XDIR); fi;) :
	$(call make-depend,$<,$@,$(subst .o,.d,$@))
	$(ECHO) Compiling: $<
	$(AT)$(CC) $(CFLAGSD) -c -o $@ $<

######################################################################### CU
$(OBJDIR)/%.o: %.cu
	$(AT)if [ ! -d $(OBJDIR) ]; then mkdir -p $(OBJDIR); fi
	$(foreach XDIR,$(EXTRA_DIRECTORIES), if [ ! -d $(OBJDIR)/$(XDIR) ]; then mkdir -p $(OBJDIR)/$(XDIR); fi;) :
	$(call make-depend,$<,$@,$(subst .o,.d,$@))
	$(ECHO) Compiling CUDA release: $<
	$(AT)$(CUCC) $(CUFLAGS) -c -o $@ $<

$(DOBJDIR)/%.o: %.cu
	$(AT)if [ ! -d $(DOBJDIR) ]; then mkdir -p $(DOBJDIR); fi
	$(foreach XDIR,$(EXTRA_DIRECTORIES), if [ ! -d $(DOBJDIR)/$(XDIR) ]; then mkdir -p $(DOBJDIR)/$(XDIR); fi;) :
	$(call make-depend,$<,$@,$(subst .o,.d,$@))
	$(ECHO) Compiling CUDA debug: $<
	$(AT)$(CUCC) $(CUFLAGSD) -c -o $@ $<

clean:
	$(ECHO) Cleaning...
	$(AT)-rm -rf $(OBJDIR) $(DOBJDIR) $(OUTDIR)/$(OUTNAME_RELEASE) $(OUTDIR)/$(OUTNAME_DEBUG)
	$(AT)-rm -rf *.engine

ifneq "$(MAKECMDGOALS)" "clean"
  -include $(OBJDIR)/*.d $(DOBJDIR)/*.d
endif

ifeq ($(DO_CUDNN_CHECK), 1)
# To display newlines in the message
define _cudnn_missing_newline_5020fd0


endef
SHELL=/bin/bash
CUDNN_CHECK = $(shell echo -e '\#include <cudnn.h>\nint main(){ cudnnCreate(nullptr); return 0; }' | $(CC) -xc++ -o /dev/null $(CFLAGS) $(LFLAGS) - $(COMMON_LIBS) 2> /dev/null && echo 'passed_cudnn_exists_check')
ifneq ($(CUDNN_CHECK), passed_cudnn_exists_check)
$(error $(_cudnn_missing_newline_5020fd0)$(_cudnn_missing_newline_5020fd0)This sample requires CUDNN, but it could not be found.$(_cudnn_missing_newline_5020fd0)Please install CUDNN from https://developer.nvidia.com/cudnn or specify CUDNN_INSTALL_DIR when compiling.$(_cudnn_missing_newline_5020fd0)For example, `make CUDNN_INSTALL_DIR=/path/to/CUDNN/` where /path/to/CUDNN/ contains include/ and lib/ subdirectories.$(_cudnn_missing_newline_5020fd0)$(_cudnn_missing_newline_5020fd0))
endif
endif
