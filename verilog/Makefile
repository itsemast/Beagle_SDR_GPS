#
# When running Vivado on a machine/OS different from where the Kiwi sources are kept,
# and those two machines are connected via a shared folder, specify the shared folder directory
# with V_SRC_DIR and use the makefile targets "cv2" and "cb" to copy sources and results back and forth.
#
# When building in batch mode, as opposed to using the Vivado UI, V_SRC_DIR is passed to the tcl script
# to specify where to place the resulting .bit files.
#
# V_SRC_DIR can be set as a global environment variable otherwise it will default to the value below.
#

# include any rules from another FPGA project using the KiwiSDR framework (i.e. CFG = other)
-include $(wildcard */Makefile.inc)

ifeq ($(V_PROJ),)
    V_PROJ = KiwiSDR
endif

ifeq ($(DEBIAN_DEVSYS),)
    DEBIAN_DEVSYS = verilog_host
endif


ifeq ($(DEBIAN_DEVSYS),$(DEVSYS))

################################
# devsys
################################

V_DIR = ~/shared/shared

$(GEN_VERILOG): $(GEN_DIR)/kiwi.gen.h verilog/rx/cic_gen.c
	(cd verilog/rx; make)

# generate the files needed to build the Verilog code
verilog: $(GEN_VERILOG)
	@echo verilog/ directory should now contain all necessary generated files:
	@echo verilog/kiwi.gen.vh, verilog/rx/cic_*.vh

# command to "copy verilog" from KiwiSDR distribution to the Vivado build location
# designed to complement the "make cv2" command run on the Vivado build machine
EXCLUDE_CV = ".DS_Store" "rx/cic_gen" "rx/*.dSYM" "*(original)*"

cv cv2: $(GEN_VERILOG)
	@echo V_PROJ = $(V_PROJ)
	@echo V_DIR = $(V_DIR)
	rsync -av --delete $(addprefix --exclude , $(EXCLUDE_CV)) verilog/ $(V_DIR)/$(V_PROJ)
	rsync -av --delete $(addprefix --exclude , $(EXCLUDE_CV)) verilog.Vivado.2017.4.ip/ $(V_DIR)/$(V_PROJ).Vivado.2017.4.ip

sum:
	sum *.bit

endif


ifeq ($(DEBIAN_DEVSYS),verilog_host)

################################
# Vivado build host
################################

ifeq ($(V_SRC_DIR)x,x)
    V_SRC_DIR = /media/sf_shared
endif

all:
	@reset
	@echo "build project with Vivado via tcl"
	time vivado -mode batch -source make_proj.tcl -tclargs --result_dir $(V_SRC_DIR)
	
rx4:
	@reset
	@echo "build rx4_wf4 project with Vivado via tcl"
	time vivado -mode batch -source make_proj.tcl -tclargs --result_dir $(V_SRC_DIR) --rx4_wf4
	
rx8:
	@reset
	@echo "build rx8_wf2 project with Vivado via tcl"
	time vivado -mode batch -source make_proj.tcl -tclargs --result_dir $(V_SRC_DIR) --rx8_wf2
	
rx3:
	@reset
	@echo "build rx3_wf3 project with Vivado via tcl"
	time vivado -mode batch -source make_proj.tcl -tclargs --result_dir $(V_SRC_DIR) --rx3_wf3
	
rx14:
	@reset
	@echo "build rx14_wf0 project with Vivado via tcl"
	time vivado -mode batch -source make_proj.tcl -tclargs --result_dir $(V_SRC_DIR) --rx14_wf0
	
rebuild:
	@reset
	@echo "rebuild project ip with Vivado via tcl"
	rm -rf KiwiSDR generated/*
	mkdir -p generated
	time vivado -mode batch -source make_proj.tcl -tclargs --result_dir $(V_SRC_DIR) --regen_ip
	
# command to "copy verilog" from KiwiSDR distribution into our build directory
# designed to complement the "make cv" command run in the KiwiSDR server build directory
EXCLUDE_RSYNC = ".DS_Store" "KiwiSDR/" "generated" ".Xil/" "vivado.*"

# cv22 is a rule that can be defined in */Makefile.inc (i.e. CFG = other) to do additional source installation
cv22:

cv cv2:
	@echo V_PROJ = $(V_PROJ)
	@echo V_SRC_DIR = $(V_SRC_DIR)
	rsync -av --delete $(addprefix --exclude , $(EXCLUDE_RSYNC)) $(V_SRC_DIR)/$(V_PROJ)/ .
	rsync -av --delete --exclude .DS_Store $(V_SRC_DIR)/$(V_PROJ).Vivado.2017.4.ip/ ../import_ip
	make cv22 V_SRC_DIR=$(V_SRC_DIR)

RX_CFG = $(shell grep RX_CFG kiwi.cfg.vh | cut -d\  -f 4 | tr -d ';')
ifeq ($(RX_CFG),4)
    CFG = rx4.wf4
else ifeq ($(RX_CFG),8)
    CFG = rx8.wf2
else ifeq ($(RX_CFG),3)
    CFG = rx3.wf3
else ifeq ($(RX_CFG),14)
    CFG = rx14.wf0
else
    CFG = other
endif

cb:
	@echo "CFG =" $(CFG)
	cp ../*.runs/impl_1/KiwiSDR.bit $(V_SRC_DIR)/KiwiSDR.$(CFG).bit

sum:
	sum /media/sf_shared/*.bit

ts:
	-tail -f KiwiSDR/KiwiSDR.runs/synth_1/runme.log

cs:
	cat KiwiSDR/KiwiSDR.runs/synth_1/runme.log

ti:
	-tail -f KiwiSDR/KiwiSDR.runs/impl_1/runme.log

ci:
	cat KiwiSDR/KiwiSDR.runs/impl_1/runme.log

RSYNC_REV_INC = $(addprefix --include=, "*.veo" "*.xci" "*.xml")
cip_rev:
	rsync -av --delete-excluded --prune-empty-dirs --include "*/"  $(RSYNC_REV_INC) --exclude="*" ../KiwiSDR.srcs/sources_1/ip/ ../import_ip/
	rsync -av --delete --exclude .DS_Store ../import_ip/ $(V_SRC_DIR)/KiwiSDR.ip

cm:
	cp kiwi_map.mrp kiwi_map_mrp.html $(V_SRC_DIR)
	cp KiwiSDR.syr kiwi_syr.html $(V_SRC_DIR)
	cp KiwiSDR.twr kiwi_twr.html $(V_SRC_DIR)
	cp KiwiSDR.twx $(V_SRC_DIR)

cp:
	cp KiwiSDR.pwr $(V_SRC_DIR)

clean:
	rm -f kiwi.coe

endif
