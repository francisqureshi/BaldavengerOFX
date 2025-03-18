PLUGINNAME = Convolution

INSTALL_PLUGIN := TRUE

UNAME_SYSTEM := $(shell uname -s)
BUILD_DIR = build

# ifeq ($(UNAME_SYSTEM), Darwin)
# ENABLEMETAL := TRUE
# OS = macOS
# GPU = METAL
# # Add architecture flags for Universal Binary
# ARCH_FLAGS = -arch arm64 -arch x86_64
# # Add these flags to both compilation and linking
# CXXFLAGS += $(ARCH_FLAGS)
# LDFLAGS += $(ARCH_FLAGS) -framework Metal -framework AppKit
# BUNDLE_DIR = $(BUILD_DIR)/$(PLUGINNAME).ofx.bundle/Contents/MacOS/
# endif

# ifeq ($(UNAME_SYSTEM), Linux)
# ENABLECUDA := TRUE
# OS = CentOS
# GPU = CUDA
# CUDAPATH ?= /usr/local/cuda
# NVCC = ${CUDAPATH}/bin/nvcc
# NVCCFLAGS = --compiler-options="-fPIC"
# LDFLAGS += -L${CUDAPATH}/lib64 -lcuda -lcudart_static
# BUNDLE_DIR = $(BUILD_DIR)/$(PLUGINNAME).ofx.bundle/Contents/Linux-x86-64/
# endif

ifeq ($(UNAME_SYSTEM), Linux)
	ENABLECUDA := TRUE
	OS = CentOS
	GPU = CUDA
	AMDAPP_PATH ?= /opt/AMDAPP
	CXXFLAGS += -I${AMDAPP_PATH}/include -fPIC
	CUDAPATH ?= /usr/local/cuda
	NVCC = ${CUDAPATH}/bin/nvcc
	NVCCFLAGS = --compiler-options="-fPIC"
	LDFLAGS = -shared -fvisibility=hidden -L${CUDAPATH}/lib64 -lcuda -lcudart_static
	BUNDLE_DIR = $(BUILD_DIR)/$(PLUGINNAME).ofx.bundle/Contents/Linux-x86-64/
	CUDA_OBJ = CudaKernel.o
else
	ENABLEMETAL := TRUE
	OS = macOS
	GPU = METAL
	ARCH_FLAGS = -arch arm64 -arch x86_64
	CXXFLAGS += ${ARCH_FLAGS}
	LDFLAGS = -bundle -fvisibility=hidden -F/Library/Frameworks -framework OpenCL -framework Metal -framework AppKit
	LDFLAGS += ${ARCH_FLAGS}
	BUNDLE_DIR = $(BUILD_DIR)/$(PLUGINNAME).ofx.bundle/Contents/MacOS/
	METAL_OBJ = MetalKernel.o
endif

DR_VERSION = DR19
CUDA_VERSION = 11.0

PLUGINOBJECTSCUDA = $(PLUGINNAME)PluginCUDA.o $(PLUGINNAME)CudaKernel.o
PLUGINOBJECTSMETAL = $(PLUGINNAME)PluginMETAL.o $(PLUGINNAME)MetalKernel.o
RESOURCES = BaldavengerOFX.$(PLUGINNAME).png

ifeq ($(ENABLECUDA), TRUE)
PLUGINOBJECTS = $(PLUGINOBJECTSCUDA)
endif

ifeq ($(ENABLEMETAL), TRUE)
PLUGINOBJECTS = $(PLUGINOBJECTSMETAL)
endif

PLUGINNAME_LG = $(PLUGINNAME).$(OS).$(DR_VERSION).$(GPU)

# OpenFX Support Library objects
OFX_SUPPORT_OBJECTS = ofxsCore.o ofxsImageEffect.o ofxsInteract.o ofxsLog.o ofxsMultiThread.o ofxsParams.o ofxsProperty.o ofxsPropertyValidation.o

all: $(BUILD_DIR)/$(PLUGINNAME).ofx

$(BUILD_DIR)/$(PLUGINNAME).ofx: $(PLUGINOBJECTS) $(OFX_SUPPORT_OBJECTS)
	@mkdir -p $(BUILD_DIR)
	$(CXX) $^ -o $@ $(LDFLAGS)
	@mkdir -p $(BUNDLE_DIR)
	cp $(BUILD_DIR)/$(PLUGINNAME).ofx $(BUNDLE_DIR)/$(PLUGINNAME).$(OS).$(DR_VERSION).$(GPU).ofx
	@if [ -d Resources ]; then \
		mkdir -p $(BUILD_DIR)/$(PLUGINNAME).ofx.bundle/Contents/Resources; \
		cp Resources/* $(BUILD_DIR)/$(PLUGINNAME).ofx.bundle/Contents/Resources/; \
	fi

# Specific rule for Metal kernel compilation
$(PLUGINNAME)MetalKernel.o: $(PLUGINNAME)MetalKernel.mm
	$(CXX) -c $< $(CXXFLAGS)

# Specific rule for CUDA kernel compilation
$(PLUGINNAME)CudaKernel.o: $(PLUGINNAME)CudaKernel.cu
	$(NVCC) -c $< $(NVCCFLAGS)

# Rule for OpenFX Support Library
%.o: ../openfx/Support/Library/%.cpp
	$(CXX) -c $< $(CXXFLAGS) -I../openfx/include -I../openfx/Support/include

# Clean target
clean:
	rm -f *.o
	rm -rf $(BUILD_DIR)

# Install target
install: $(BUILD_DIR)/$(PLUGINNAME).ofx
	@mkdir -p /Library/OFX/Plugins/$(PLUGINNAME).ofx.bundle/Contents/MacOS
	cp -R $(BUILD_DIR)/$(PLUGINNAME).ofx.bundle/* /Library/OFX/Plugins/$(PLUGINNAME).ofx.bundle/

TOP_SRCDIR = ..
include $(TOP_SRCDIR)/Makefile.master