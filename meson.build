# Copyright lowRISC contributors.
# Licensed under the Apache License, Version 2.0, see LICENSE for details.
# SPDX-License-Identifier: Apache-2.0

project(
  'opentitan', 'c', 'cpp',
  version: '0.1',
  meson_version: '>=0.51.0', # Matches version in python-requirements.txt
  default_options: [
    'c_std=c11',
    'build.c_std=c11',
    'cpp_std=c++14',
    'build.cpp_std=c++14',
    'warning_level=1',
    'werror=true',
    'debug=true',
    'b_staticpic=false', # Disable PIC for device static libraries
    'b_pie=false',       # Disable PIE for device executables
  ],
)

ot_version = get_option('ot_version')
if ot_version == 'undef'
  error('ot_version option not set. Please run meson with a valid OpenTitan version option.')
endif

dev_bin_dir = get_option('dev_bin_dir')
host_bin_dir = get_option('host_bin_dir')
if dev_bin_dir == 'undef' or host_bin_dir == 'undef'
  error('dev_bin_dir option not set. Please run meson with a valid binary directory option.')
endif
tock_local = get_option('tock_local')

# See the comment in `./util/meson-purge-includes.sh` for why this is necessary.
if not get_option('keep_includes')
  meson.add_postconf_script(meson.source_root() + '/util/meson-purge-includes.sh')
endif


# C Arguments to optimize for size, used on cross builds only.
optimize_size_args = [
  '-Os', # General "Optimize for Size" Option
  '-fvisibility=hidden', # Hide symbols by default
]

if meson.get_compiler('c', native: false).has_argument('-Wa,--no-pad-sections')
    # Don't pad assembly sections. This was originally added to avoid sections
    # being padded to the alignment size. Specifically, .vectors was being
    # padded to 256 bytes when aligning to that value, when it only needed to be
    # 128 bytes long. Clang doesn't do this padding, so restricting this option
    # to GCC doesn't waste space when compiling with Clang.
    optimize_size_args += '-Wa,--no-pad-sections'
endif

# The following flags are applied to *all* builds, both cross builds and native
# builds.
c_cpp_args = [
  # We use absolute include paths as much as possible.
  '-I' + meson.source_root(),
  '-I' + meson.build_root(),
]
add_project_arguments(c_cpp_args, language: ['c', 'cpp'], native: false)
add_project_arguments(c_cpp_args, language: ['c', 'cpp'], native: true)

# The following flags are applied only to cross builds
c_cpp_cross_args = [
  # Do not use standard system headers
  '-nostdinc',
  # Use OpenTitan's freestanding headers instead
  '-isystem' + meson.source_root() / 'sw/device/lib/base/freestanding',
]
add_project_arguments(
  c_cpp_cross_args,
  optimize_size_args,
  language: ['c', 'cpp'], native: false)

# The following flags are applied only to cross builds
c_cpp_cross_link_args = [
  # Do not use standard system startup files or libraries
  '-nostartfiles',
  '-nostdlib',
   # Only link static files
  '-static',
]
add_project_link_arguments(
  c_cpp_cross_link_args,
  language: ['c', 'cpp'], native: false)


# Common program references.
prog_python = import('python').find_installation('python3')
prog_objdump = find_program('objdump')
prog_objcopy = find_program('objcopy')
prog_srec_cat = find_program('srec_cat')
prog_git = find_program('git')

# Hardware register headers. These are generated from HJSON files, and accesible
# in C via |#include "{IP_NAME}_regs.h"|.
gen_hw_hdr = generator(
  prog_python,
  output: '@BASENAME@_regs.h',
  arguments: [
    '@SOURCE_DIR@/util/regtool.py', '-D', '-o', '@BUILD_DIR@/@BASENAME@_regs.h',
    '@INPUT@',
  ],
)

# TODO: Considering moving these into hw/ip directories.
hw_ip_aes_reg_h = gen_hw_hdr.process('hw/ip/aes/data/aes.hjson')
hw_ip_flash_ctrl_reg_h = gen_hw_hdr.process('hw/ip/flash_ctrl/data/flash_ctrl.hjson')
hw_ip_gpio_reg_h = gen_hw_hdr.process('hw/ip/gpio/data/gpio.hjson')
hw_ip_hmac_reg_h = gen_hw_hdr.process('hw/ip/hmac/data/hmac.hjson')
hw_ip_i2c_reg_h = gen_hw_hdr.process('hw/ip/i2c/data/i2c.hjson')
hw_ip_otbn_reg_h = gen_hw_hdr.process('hw/ip/otbn/data/otbn.hjson')
hw_ip_spi_device_reg_h = gen_hw_hdr.process('hw/ip/spi_device/data/spi_device.hjson')
hw_ip_rv_timer_reg_h = gen_hw_hdr.process('hw/ip/rv_timer/data/rv_timer.hjson')
hw_ip_uart_reg_h = gen_hw_hdr.process('hw/ip/uart/data/uart.hjson')
hw_ip_usbdev_reg_h = gen_hw_hdr.process('hw/ip/usbdev/data/usbdev.hjson')
hw_top_earlgrey_pinmux_reg_h = gen_hw_hdr.process('hw/top_earlgrey/ip/pinmux/data/autogen/pinmux.hjson')
hw_top_earlgrey_rv_plic_reg_h = gen_hw_hdr.process('hw/top_earlgrey/ip/rv_plic/data/autogen/rv_plic.hjson')

# Top Earlgrey library (top_earlgrey)
# The sources for this are generated into the hw hierarchy.
top_earlgrey = declare_dependency(
  link_with: static_library(
    'top_earlgrey_ot',
    sources: [
      'hw/top_earlgrey/sw/autogen/top_earlgrey.c',
    ],
  )
)

subdir('sw')
