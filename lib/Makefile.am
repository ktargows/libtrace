lib_LTLIBRARIES = libtrace.la
include_HEADERS = libtrace.h dagformat.h lt_inttypes.h daglegacy.h \
        rt_protocol.h erftypes.h libtrace_parallel.h \
        data-struct/ring_buffer.h data-struct/object_cache.h \
        data-struct/vector.h data-struct/message_queue.h \
        data-struct/deque.h data-struct/linked_list.h \
        data-struct/sliding_window.h hash_toeplitz.h \
        data-struct/buckets.h

AM_CFLAGS=@LIBCFLAGS@ @CFLAG_VISIBILITY@ -pthread
AM_CXXFLAGS=@LIBCXXFLAGS@ @CFLAG_VISIBILITY@ -pthread

extra_DIST = format_template.c
NATIVEFORMATS=format_linux_common.c format_linux_ring.c format_linux_int.c format_linux_common.h
BPFFORMATS=format_bpf.c

if HAVE_DAG
if DAG2_4
DAGSOURCE=format_dag24.c
else
DAGSOURCE=format_dag25.c
endif
else
DAGSOURCE=
endif
EXTRA_DIST=format_dag24.c format_dag25.c dpdk_libtrace.mk

if HAVE_LLVM
BPFJITSOURCE=bpf-jit/bpf-jit.cc
else
BPFJITSOURCE=
endif

if HAVE_DPDK
NATIVEFORMATS+= format_dpdk.c
# So we also make libtrace.mk in dpdk otherwise automake tries to expand
# it too early which I cannot seem to stop unless we use a path that
# doesn't exist currently
export RTE_SDK=@RTE_SDK@
export RTE_TARGET=@RTE_TARGET@
export SAVED_CFLAGS:=$(CFLAGS)
export SAVED_CXXFLAGS:=$(CXXFLAGS)
include $(RTE_SDK)/mk/rte.vars.mk
# We need to add -Wl before the linker otherwise this breaks our build
# And include any libraries that DPDK might depend upon
export DPDK_LIBTRACE_MK=dpdk_libtrace.mk
include $(DPDK_LIBTRACE_MK)
export CFLAGS += $(SAVED_CFLAGS)
export CXXFLAGS += $(SAVED_CXXFLAGS)
endif

libtrace_la_SOURCES = trace.c trace_parallel.c common.h \
		format_erf.c format_pcap.c format_legacy.c \
		format_rt.c format_helper.c format_helper.h format_pcapfile.c \
		format_duck.c format_tsh.c $(NATIVEFORMATS) $(BPFFORMATS) \
		format_atmhdr.c \
		libtrace_int.h lt_inttypes.h lt_bswap.h \
		linktypes.c link_wireless.c byteswap.c \
		checksum.c checksum.h \
		protocols_pktmeta.c protocols_l2.c protocols_l3.c \
		protocols_transport.c protocols.h protocols_ospf.c \
		protocols_application.c \
		$(DAGSOURCE) format_erf.h \
		$(BPFJITSOURCE) \
		libtrace_arphrd.h \
		data-struct/ring_buffer.c data-struct/vector.c \
		data-struct/message_queue.c data-struct/deque.c \
		data-struct/sliding_window.c data-struct/object_cache.c \
		data-struct/linked_list.c hash_toeplitz.c combiner_ordered.c \
                data-struct/buckets.c \
		combiner_sorted.c combiner_unordered.c \
		pthread_spinlock.c pthread_spinlock.h

if DAG2_4
nodist_libtrace_la_SOURCES = dagopts.c dagapi.c
endif

AM_CPPFLAGS= @ADD_INCLS@
libtrace_la_LIBADD = @LIBTRACE_LIBS@ @LTLIBOBJS@ $(DPDKLIBS)
libtrace_la_LDFLAGS=-version-info @LIBTRACE_MAJOR@:@LIBTRACE_MINOR@:@LIBTRACE_MID@ @ADD_LDFLAGS@
dagapi.c:
	cp @DAG_TOOLS_DIR@/dagapi.c .

dagopts.c:
	cp @DAG_TOOLS_DIR@/dagopts.c .

if HAVE_LLVM
# Rules to build the llvm .cc file from a C description of each opcode.
bpf-jit/bpf-opcodes.llvm.bc: bpf-jit/bpf-opcodes.c
	$(LLVM_GCC) -std=c99 -c -O0 -emit-llvm $< -o $@

bpf-jit/bpf-opcodes.llvm.cc: bpf-jit/bpf-opcodes.llvm.bc
	`$(LLVM_CONFIG) --bindir`/llc -march=cpp -cppgen=module $< -o $@

bpf-jit/bpf-jit.cc: bpf-jit/bpf-opcodes.llvm.cc

EXTRA_DIST+=bpf-jit/bpf-opcodes.c
BUILT_SOURCES=bpf-jit/bpf-opcodes.llvm.cc
endif
CLEANFILES=bpf-jit/bpf-opcodes.llvm.cc bpf-jit/bpf-opcodes.llvm.bc
