softmmu_ss.add(pixman)
specific_ss.add(when: ['CONFIG_SOFTMMU'], if_true: pixman)   # for the include path

softmmu_ss.add(files(
  'clipboard.c',
  'console.c',
  'cursor.c',
  'input-keymap.c',
  'input-legacy.c',
  'input-barrier.c',
  'input.c',
  'kbd-state.c',
  'keymaps.c',
  'qemu-pixman.c',
  'udmabuf.c',
))

xemu_ss = ss.source_set()
xemu_ss.add(files(
  'imgui/imgui.cpp',
  'imgui/imgui_draw.cpp',
  'imgui/imgui_tables.cpp',
  'imgui/imgui_demo.cpp',
  'imgui/imgui_widgets.cpp',
  'imgui/backends/imgui_impl_opengl3.cpp',
  'imgui/backends/imgui_impl_sdl.cpp',
  'implot/implot.cpp',
  'implot/implot_demo.cpp',
  'implot/implot_items.cpp',

  'xemu.c',
  'xemu-custom-widgets.c',
  'xemu-data.c',
  'xemu-input.c',
  'xemu-monitor.c',
  'xemu-net.c',
  'xemu-settings.c',
  'xemu-shaders.c',
  'xemu-hud.cc',
  'xemu-reporting.cc',
))
xemu_ss.add(when: 'CONFIG_WIN32', if_true: files('xemu-update.cc'))

imgui_flags = ['-DIMGUI_IMPL_OPENGL_LOADER_CUSTOM="epoxy/gl.h"']

if 'CONFIG_WIN32' in config_host
imgui_flags += ['-DWIN32', '-DMINGW32']
endif

if 'CONFIG_LINUX' in config_host
imgui_flags += ['-DLINUX']
endif

if 'CONFIG_DARWIN' in config_host
imgui_flags += ['-DAPPLE']
xemu_cocoa = dependency('appleframeworks', modules: 'Cocoa')
xemu_ss.add(xemu_cocoa) # FIXME: Use existing cocoa name
endif

imgui = declare_dependency(compile_args: imgui_flags, include_directories: 'imgui')
xemu_ss.add(sdl, opengl, openssl, imgui)
xemu_ss.add(when: 'CONFIG_LINUX', if_true: [xemu_gtk, files('xemu-os-utils-linux.c', 'noc_file_dialog_gtk.c')])
xemu_ss.add(when: 'CONFIG_WIN32', if_true: files('xemu-os-utils-windows.c', 'noc_file_dialog_win32.c'))
xemu_ss.add(when: 'CONFIG_DARWIN', if_true: files('xemu-os-utils-macos.m', 'noc_file_dialog_macos.m'))
softmmu_ss.add_all(xemu_ss)

softmmu_ss.add([spice_headers, files('spice-module.c')])
softmmu_ss.add(when: spice_protocol, if_true: files('vdagent.c'))

softmmu_ss.add(when: 'CONFIG_LINUX', if_true: files('input-linux.c'))
softmmu_ss.add(when: cocoa, if_true: files('cocoa.m'))

vnc_ss = ss.source_set()
vnc_ss.add(files(
  'vnc.c',
  'vnc-enc-zlib.c',
  'vnc-enc-hextile.c',
  'vnc-enc-tight.c',
  'vnc-palette.c',
  'vnc-enc-zrle.c',
  'vnc-auth-vencrypt.c',
  'vnc-ws.c',
  'vnc-jobs.c',
  'vnc-clipboard.c',
))
vnc_ss.add(zlib, png, jpeg, gnutls)
vnc_ss.add(when: sasl, if_true: files('vnc-auth-sasl.c'))
softmmu_ss.add_all(when: vnc, if_true: vnc_ss)
softmmu_ss.add(when: vnc, if_false: files('vnc-stubs.c'))
specific_ss.add(when: ['CONFIG_SOFTMMU'], if_true: opengl)

ui_modules = {}

xemu_enable_extra_ui_modules = false

if xemu_enable_extra_ui_modules

if curses.found()
  curses_ss = ss.source_set()
  curses_ss.add(when: [curses, iconv], if_true: [files('curses.c'), pixman])
  ui_modules += {'curses' : curses_ss}
endif

if config_host.has_key('CONFIG_OPENGL')
  opengl_ss = ss.source_set()
  opengl_ss.add(gbm)
  opengl_ss.add(when: [opengl, pixman, 'CONFIG_OPENGL'],
               if_true: files('shader.c', 'console-gl.c', 'egl-helpers.c', 'egl-context.c'))
  ui_modules += {'opengl' : opengl_ss}
endif

if config_host.has_key('CONFIG_OPENGL') and gbm.found()
  egl_headless_ss = ss.source_set()
  egl_headless_ss.add(when: [opengl, gbm, pixman, 'CONFIG_OPENGL'],
                      if_true: files('egl-headless.c'))
  ui_modules += {'egl-headless' : egl_headless_ss}
endif

if gtk.found()
  softmmu_ss.add(when: 'CONFIG_WIN32', if_true: files('win32-kbd-hook.c'))

  gtk_ss = ss.source_set()
  gtk_ss.add(gtk, vte, pixman, files('gtk.c', 'gtk-clipboard.c'))
  gtk_ss.add(when: x11, if_true: files('x_keymap.c'))
  gtk_ss.add(when: [opengl, 'CONFIG_OPENGL'], if_true: files('gtk-gl-area.c'))
  gtk_ss.add(when: [x11, opengl, 'CONFIG_OPENGL'], if_true: files('gtk-egl.c'))
  ui_modules += {'gtk' : gtk_ss}
endif

if sdl.found()
  softmmu_ss.add(when: 'CONFIG_WIN32', if_true: files('win32-kbd-hook.c'))

  sdl_ss = ss.source_set()
  sdl_ss.add(sdl, sdl_image, pixman, glib, files(
    'sdl2-2d.c',
    'sdl2-input.c',
    'sdl2.c',
  ))
  sdl_ss.add(when: [opengl, 'CONFIG_OPENGL'], if_true: files('sdl2-gl.c'))
  sdl_ss.add(when: x11, if_true: files('x_keymap.c'))
  ui_modules += {'sdl' : sdl_ss}
endif

if config_host.has_key('CONFIG_SPICE')
  spice_core_ss = ss.source_set()
  spice_core_ss.add(spice, pixman, files(
    'spice-core.c',
    'spice-input.c',
    'spice-display.c'
  ))
  ui_modules += {'spice-core' : spice_core_ss}
endif

if config_host.has_key('CONFIG_SPICE') and config_host.has_key('CONFIG_GIO')
  spice_ss = ss.source_set()
  spice_ss.add(spice, gio, pixman, files('spice-app.c'))
  ui_modules += {'spice-app': spice_ss}
endif

endif # xemu_enable_extra_ui_modules

keymap_gen = find_program('keycodemapdb/tools/keymap-gen')

keymaps = [
  ['atset1', 'qcode'],
  ['linux', 'qcode'],
  ['qcode', 'atset1'],
  ['qcode', 'atset2'],
  ['qcode', 'atset3'],
  ['qcode', 'linux'],
  ['qcode', 'qnum'],
  ['qcode', 'sun'],
  ['qnum', 'qcode'],
  ['usb', 'qcode'],
  ['win32', 'qcode'],
  ['x11', 'qcode'],
  ['xorgevdev', 'qcode'],
  ['xorgkbd', 'qcode'],
  ['xorgxquartz', 'qcode'],
  ['xorgxwin', 'qcode'],
  ['osx', 'qcode'],
]

if have_system or xkbcommon.found()
  foreach e : keymaps
    output = 'input-keymap-@0@-to-@1@.c.inc'.format(e[0], e[1])
    genh += custom_target(output,
                  output: output,
                  capture: true,
                  input: files('keycodemapdb/data/keymaps.csv'),
                  command: [python.full_path(), files('keycodemapdb/tools/keymap-gen'),
                            'code-map',
                            '--lang', 'glib2',
                            '--varname', 'qemu_input_map_@0@_to_@1@'.format(e[0], e[1]),
                            '@INPUT0@', e[0], e[1]])
  endforeach
endif

subdir('shader')

if have_system
  subdir('icons')

  install_data('qemu.desktop', install_dir: qemu_desktopdir)
endif

modules += {'ui': ui_modules}
