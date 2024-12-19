
PYTHON_VENV = venv
VIVADO_LOCATION ?= /tools/Xilinx/Vivado
VIVADO_VERSION ?= $$(ls -d $(VIVADO_LOCATION)/*.* | tr / \\n | tail -n 1)
RISCV_TOOLCHAIN = riscv64-unknown-elf-gcc-10.1.0-2020.08.2-x86_64-linux-ubuntu14
PYTHON_FLAGS =
BUILD_TARGET ?= digilent_nexys_video.py
BUILD_FLAGS =


.PHONY: env.sh build load


# Creating python virtual environment.
$(PYTHON_VENV):
	python3 -m venv $(PYTHON_VENV)
	
venv: $(PYTHON_VENV)

# Downloading and extracting RISCV Toolchain
$(RISCV_TOOLCHAIN):
	curl -fSL# https://static.dev.sifive.com/dev-tools/freedom-tools/v2020.08/$(RISCV_TOOLCHAIN).tar.gz > $(RISCV_TOOLCHAIN).tar.gz
	tar -xzf $(RISCV_TOOLCHAIN).tar.gz --checkpoint=.10000

toolchain: $(RISCV_TOOLCHAIN)

# Source settings64.sh (vivado) and virtual environment in one script.
env: $(PYTHON_VENV) $(RISCV_TOOLCHAIN)
	echo 'alias source=.' > ./env.sh
	echo '. $(PYTHON_VENV)/bin/activate' >> ./env.sh
	echo ". $(VIVADO_LOCATION)/$(VIVADO_VERSION)/settings64.sh" >> ./env.sh
	echo 'export PATH="$$PATH:$$(pwd)/$(RISCV_TOOLCHAIN)/bin"' >> ./env.sh

# Install dependencies.
$(PYTHON_VENV)/dependencies_installed.txt: | env
	git submodule update --init --recursive dependencies
	. ./env.sh && pip install -r ./requirements.txt
	@echo Installed dependencies on $$(date). > $(PYTHON_VENV)/dependencies_installed.txt

deps: $(PYTHON_VENV)/dependencies_installed.txt

# Build SoC.
build: deps | env
	. ./env.sh; \
	python $(PYTHON_FLAGS) ./dependencies/litex-boards/litex_boards/targets/$(BUILD_TARGET) --build $(BUILD_FLAGS)

# Load Design
load: | env
	. ./env.sh; \
	python $(PYTHON_FLAGS) ./dependencies/litex-boards/litex_boards/targets/$(BUILD_TARGET) --load

clean:
	git submodule deinit -f dependencies
	rm -r $(PYTHON_VENV) && rm -r build
	