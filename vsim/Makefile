base_dir=$(abspath ..)
sim_dir=$(abspath .)

include $(base_dir)/Makefrag-variables

sim = $(sim_dir)/simv-$(PROJECT)-$(CONFIG)
sim_debug = $(sim_dir)/simv-$(PROJECT)-$(CONFIG)-debug

simv = $(sim_dir)/simv-$(PROJECT)-$(CONFIG)
simv_debug = $(sim_dir)/simv-$(PROJECT)-$(CONFIG)-debug

default: $(simv)

debug: $(simv_debug)

include $(base_dir)/Makefrag

ifneq ($(filter run% %.run %.out %.vpd %.vcd,$(MAKECMDGOALS)),)
-include $(build_dir)/$(long_name).d
endif

VCS = vcs -full64

VCS_OPTS = -notice -line +lint=all,noVCDE,noONGS,noUI -error=PCWM-L -timescale=1ns/10ps -quiet \
	+rad +v2k +vcs+lic+wait \
	+vc+list -CC "-I$(VCS_HOME)/include" \
	-CC "-I$(RISCV)/include -I$(base_dir)/testchipip/csrc" \
	-CC "-std=c++11" \
	-CC "-Wl,-rpath,$(RISCV)/lib" \
	$(RISCV)/lib/libfesvr.so \
	-sverilog \
	+incdir+$(build_dir) \
	+define+CLOCK_PERIOD=1.0 $(sim_vsrcs) $(sim_csrcs) \
	+define+PRINTF_COND=$(TB).printf_cond \
	+define+STOP_COND=!$(TB).reset \
	+define+RANDOMIZE_MEM_INIT \
	+define+RANDOMIZE_REG_INIT \
	+define+RANDOMIZE_GARBAGE_ASSIGN \
	+define+RANDOMIZE_INVALID_ASSIGN \
	+libext+.v \

VCS_OPTS += +vpi
VCS_OPTS += -CC "-DVCS_VPI"

verilog: $(sim_vsrcs)

$(simv): $(sim_vsrcs) $(sim_csrcs)
	rm -rf csrc && $(VCS) $(VCS_OPTS) -o $@ \
	-debug_pp

$(simv_debug) : $(sim_vsrcs) $(sim_csrcs)
	rm -rf csrc && $(VCS) $(VCS_OPTS) -o $@ \
	+define+DEBUG -debug_pp

exec_sim=$(sim) +permissive -q +ntb_random_seed_automatic
exec_sim_debug=$(sim_debug) +permissive -q +ntb_random_seed_automatic

clean:
	rm -rf $(build_dir) csrc simv-* ucli.key vc_hdrs.h

.PHONY: clean default debug
