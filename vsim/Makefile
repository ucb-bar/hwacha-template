base_dir=$(abspath ..)
sim_dir=$(abspath .)

include $(base_dir)/Makefrag-variables

sim_name = vcs

sim = $(sim_dir)/simv-$(PROJECT)-$(CONFIG)
sim_debug = $(sim_dir)/simv-$(PROJECT)-$(CONFIG)-debug

default: $(sim)

debug: $(sim_debug)

include $(base_dir)/Makefrag
ifneq ($(filter run% %.run %.out %.vpd %.vcd,$(MAKECMDGOALS)),)
-include $(build_dir)/$(long_name).d
endif

VCS = vcs -full64

VCS_OPTS = -notice -line +lint=all,noVCDE,noONGS,noUI -error=PCWM-L -timescale=1ns/10ps -quiet \
	+rad +v2k +vcs+lic+wait \
	+vc+list -CC "-I$(VCS_HOME)/include" \
	-CC "-I$(RISCV)/include" \
	-CC "-std=c++11" \
	-CC "-Wl,-rpath,$(RISCV)/lib" \
	-f $(sim_top_blackboxes) -f $(sim_harness_blackboxes) -f $(sim_dotf) \
	$(RISCV)/lib/libfesvr.so \
	-sverilog \
	+incdir+$(build_dir) \
	+define+CLOCK_PERIOD=1.0 $(sim_vsrcs) \
	+define+PRINTF_COND=$(TB).printf_cond \
	+define+STOP_COND=!$(TB).reset \
	+define+RANDOMIZE_MEM_INIT \
	+define+RANDOMIZE_REG_INIT \
	+define+RANDOMIZE_GARBAGE_ASSIGN \
	+define+RANDOMIZE_INVALID_ASSIGN \
	+define+RANDOMIZE_DELAY=2 \
	+libext+.v \

verilog: $(sim_vsrcs)

$(sim): $(sim_vsrcs) $(sim_dotf)
	rm -rf csrc && $(VCS) $(VCS_OPTS) -o $@ \
	-debug_pp

$(sim_debug) : $(sim_vsrcs) $(sim_dotf)
	rm -rf csrc && $(VCS) $(VCS_OPTS) -o $@ \
	+define+DEBUG -debug_pp

exec_sim=$(sim) +permissive -q +ntb_random_seed_automatic
exec_sim_debug=$(sim_debug) +permissive -q +ntb_random_seed_automatic

$(output_dir)/%.vpd: $(output_dir)/% $(sim_debug)
	cd $(sim_dir) && $(exec_sim_debug) +verbose +vcdplusfile=$@ +max-cycles=$(timeout_cycles) $(perm_off) $< $(disasm) $(patsubst %.vpd,%.out,$@) && [ $$PIPESTATUS -eq 0 ]

$(output_dir)/%.saif: $(output_dir)/% $(sim_debug)
	cd $(sim_dir) && rm -f $(output_dir)/pipe-$*.vcd && vcd2saif -input $(output_dir)/pipe-$*.vcd -pipe "$(exec_sim_debug) +verbose +vcdfile=$(output_dir)/pipe-$*.vcd +max-cycles=$(bmark_timeout_cycles) $(perm_off) $<" -output $@ > $(patsubst %.saif,%.out,$@) 2>&1

clean:
	rm -rf $(build_dir) csrc simv-* ucli.key vc_hdrs.h

.PHONY: clean default debug
