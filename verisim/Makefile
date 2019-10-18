base_dir=$(abspath ..)
sim_dir=$(abspath .)

include $(base_dir)/Makefrag-variables
sim_name = verilator

sim = $(sim_dir)/simulator-$(PROJECT)-$(CONFIG)
sim_debug = $(sim_dir)/simulator-$(PROJECT)-$(CONFIG)-debug
tether_file = $(build_dir)/$(CONFIG).tether

default: $(sim)

debug: $(sim_debug)

CXXFLAGS := $(CXXFLAGS) -O1 -std=c++11 -I$(RISCV)/include -D__STDC_FORMAT_MACROS
LDFLAGS := $(LDFLAGS) -L$(RISCV)/lib -Wl,-rpath,$(RISCV)/lib -L$(sim_dir) -lfesvr -lpthread

include $(base_dir)/Makefrag
include $(sim_dir)/Makefrag-verilator
ifneq ($(filter run% %.run %.out %.vpd %.vcd,$(MAKECMDGOALS)),)
-include $(build_dir)/$(long_name).d
endif

model_dir = $(build_dir)/$(long_name)
model_dir_debug = $(build_dir)/$(long_name).debug

model_header = $(model_dir)/V$(MODEL).h
model_header_debug = $(model_dir_debug)/V$(MODEL).h

model_mk = $(model_dir)/V$(MODEL).mk
model_mk_debug = $(model_dir_debug)/V$(MODEL).mk

$(model_mk): $(sim_vsrcs) $(sim_dotf) $(INSTALLED_VERILATOR)
	rm -rf $(build_dir)/$(long_name)
	mkdir -p $(build_dir)/$(long_name)
	$(VERILATOR) $(VERILATOR_FLAGS) -Mdir $(build_dir)/$(long_name) \
	-o $(sim) $(sim_vsrcs) -f $(sim_dotf) -f $(sim_top_blackboxes) -f $(sim_harness_blackboxes) -LDFLAGS "$(LDFLAGS)" \
	-CFLAGS "-I$(build_dir) -include $(build_dir)/$(long_name).plusArgs -include $(model_header)"
	touch $@

$(sim): $(model_mk)
	$(MAKE) VM_PARALLEL_BUILDS=1 -C $(build_dir)/$(long_name) -f V$(MODEL).mk


$(model_mk_debug): $(sim_vsrcs) $(sim_dotf) $(INSTALLED_VERILATOR)
	rm -rf $(build_dir)/$(long_name)
	mkdir -p $(build_dir)/$(long_name).debug
	$(VERILATOR) $(VERILATOR_FLAGS) -Mdir $(build_dir)/$(long_name).debug --trace \
	-o $(sim_debug) $(sim_vsrcs) -f $(sim_dotf) -f $(sim_top_blackboxes) -f $(sim_harness_blackboxes) -LDFLAGS "$(LDFLAGS)" \
	-CFLAGS "-I$(build_dir) -include $(build_dir)/$(long_name).plusArgs -include $(model_header_debug)"
	touch $@

$(sim_debug): $(model_mk_debug)
	$(MAKE) VM_PARALLEL_BUILDS=1 -C $(build_dir)/$(long_name).debug -f V$(MODEL).mk


exec_sim = $(sim)
exec_sim_debug = $(sim_debug)

$(output_dir)/%.vcd: $(output_dir)/% $(sim_debug)
	cd $(sim_dir) && $(exec_sim_debug) +verbose -v$@ +max-cycles=$(timeout_cycles) $(perm_off) $< $(disasm) $(patsubst %.vcd,%.out,$@) && [ $$PIPESTATUS -eq 0 ]

$(output_dir)/%.vpd: $(output_dir)/% $(sim_debug)
	if command -v vcd2vpd > /dev/null 2>&1 ; then rm -f $<.vcd && mkfifo $<.vcd && { vcd2vpd $<.vcd $@ & } fi && \
	cd $(sim_dir) && $(exec_sim_debug) +verbose -v$<.vcd +max-cycles=$(timeout_cycles) $(perm_off) $< $(disasm) $(patsubst %.vpd,%.out,$@) && [ $$PIPESTATUS -eq 0 ]
	touch $@

clean:
	rm -rf $(build_dir) ./simulator-*

.PHONY: clean default debug
