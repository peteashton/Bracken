CXXFLAGS := -c -g -O3 -pedantic -std=c++11

UNAME_S := $(shell uname -s)

# make's built-in default for CXX is "c++", so only pick a compiler when the
# user hasn't set one (e.g. `make CXX=g++-14` still works as expected).
ifeq ($(origin CXX),default)
  ifeq ($(UNAME_S),Darwin)
    CXX := clang++
  else
    CXX := g++
  endif
endif

# Detect clang by what the compiler reports, not by its name: on macOS
# "c++", "g++" and "clang++" are all Apple clang unless GCC is installed.
IS_CLANG := $(shell $(CXX) --version 2>/dev/null | grep -c -i clang)

ifneq ($(IS_CLANG),0)
  ifeq ($(UNAME_S),Darwin)
    # Apple clang has no bundled OpenMP runtime: it needs libomp, which
    # Homebrew installs keg-only (not on the default search paths).
    # Override with `make LIBOMP_PREFIX=/path/to/libomp` if needed.
    LIBOMP_PREFIX ?= $(shell brew --prefix libomp 2>/dev/null)
    OMP_DIR := $(firstword $(foreach d,$(LIBOMP_PREFIX) /opt/homebrew/opt/libomp /usr/local/opt/libomp,$(if $(wildcard $(d)/include/omp.h),$(d))))
    ifeq ($(OMP_DIR),)
      $(error OpenMP runtime (libomp) not found. Install it with `brew install libomp`, or run `make LIBOMP_PREFIX=/path/to/libomp`)
    endif
    CXXFLAGS += -Xpreprocessor -fopenmp -I$(OMP_DIR)/include
    LDFLAGS += -L$(OMP_DIR)/lib -lomp
  else
    CXXFLAGS += -Xpreprocessor -fopenmp
    LDFLAGS += -lomp
  endif
else
  CXXFLAGS += -fopenmp
  LDFLAGS += -lgomp
endif

all: kmer2read_distr

kmer2read_distr: kmer2read_distr.o ctime.o taxonomy.o kraken_processing.o
	$(CXX) -o $@ $^ $(LDFLAGS)

clean:
	rm -f *.o

%.o: %.cpp
	$(CXX) $(CXXFLAGS) $<
