#
# Video output modules
#

# Declared here, as its used by both the
# libplacebo and opengl subfolder build file.
if have_win_store
    # even if it's there, don't use it
    missing_win_glew = true
elif host_system == 'windows'
    missing_win_glew = not cdata.has('HAVE_GL_WGLEW_H')
else
    missing_win_glew = false
endif

if missing_win_glew
    # Our OpenGL code requires glew.h to be present
    opengl_dep = disabler()
else
    opengl_dep = dependency('gl', required: false)
endif
opengles2_dep = dependency('glesv2', required: get_option('gles2'))

xkbcommon_dep = dependency('xkbcommon', required: false)

if host_system == 'darwin'
    subdir('apple')
endif

subdir('libplacebo')
subdir('opengl')

if host_system == 'windows'
    subdir('win32')
endif

# Dummy video output
vlc_modules += {
    'name' : 'vdummy',
    'sources' : files('vdummy.c')
}

# Dummy window provider
vlc_modules += {
    'name' : 'wdummy',
    'sources' : files('wdummy.c')
}

# Video splitter
vlc_modules += {
    'name' : 'video_splitter',
    'sources' : files('splitter.c')
}

# YUV video output
vlc_modules += {
    'name' : 'yuv',
    'sources' : files('yuv.c')
}

# Flaschen video output
vlc_modules += {
    'name' : 'flaschen',
    'sources' : files('flaschen.c'),
    'dependencies' : [socket_libs]
}

# vmem
vlc_modules += {
    'name' : 'vmem',
    'sources' : files('vmem.c')
}

# wextern
vlc_modules += {
    'name' : 'wextern',
    'sources' : files('wextern.c')
}

# vgl
vlc_modules += {
    'name' : 'vgl',
    'sources' : files('vgl.c')
}

vlc_modules += {
    'name' : 'decklinkoutput',
    'sources' : files('decklink.cpp',
                    '../stream_out/sdi/Ancillary.cpp',
                    '../stream_out/sdi/Ancillary.hpp',
                    '../stream_out/sdi/DBMHelper.cpp',
                    '../stream_out/sdi/DBMHelper.hpp',
                    '../stream_out/sdi/SDIGenerator.cpp',
                    '../stream_out/sdi/SDIGenerator.hpp',
                    '../stream_out/sdi/V210.cpp',
                    '../stream_out/sdi/V210.hpp'),
    'dependencies' : [dl_lib, decklink_dep],
    'cpp_args' : decklink_cpp_args,
    'enabled' : decklink_dep.found(),
}

# Kernel Mode Setting
drm_dep = dependency('libdrm', version: '>= 2.4.83', required: get_option('drm'))
if drm_dep.found()
    vlc_modules += {
        'name' : 'kms',
        'sources' : files('kms.c'),
        'dependencies' : [drm_dep]
    }
endif

# Coloured ASCII art (Caca)
caca_dep = dependency('caca', version: '>= 0.99.beta14', required: get_option('caca'))
caca_deps = [ caca_dep ]
if not (host_system in ['windows', 'darwin'])
    caca_deps += x11_dep
endif
if caca_dep.found()
    vlc_modules += {
        'name' : 'caca',
        'sources' : files('caca.c'),
        'dependencies' : [caca_deps],
    }
endif

#
# x11 video outputs
#

if xcb_dep.found()
    vlc_xcb_events_lib = library('vlc_xcb_events',
        files('xcb/events.c'),
        include_directories: [vlc_include_dirs],
        dependencies: [xcb_dep, libvlccore_dep])

    if xcb_randr_dep.found() and xproto_dep.found()
        vlc_modules += {
            'name' : 'xcb_window',
            'sources' : files('xcb/window.c'),
            'dependencies' : [xcb_dep, xcb_randr_dep, xproto_dep]
        }
    endif

    if xcb_render_dep.found() and xcb_shm_dep.found()
        vlc_modules += {
            'name' : 'xcb_render',
            'sources' : files(
              'xcb/pictures.c',
              'xcb/render.c'
            ),
            'dependencies' : [xcb_dep, xcb_render_dep, xcb_shm_dep, m_lib],
            'link_with' : [vlc_xcb_events_lib]
        }
    endif

    if xcb_shm_dep.found()
        vlc_modules += {
            'name' : 'xcb_x11',
            'sources' : files(
              'xcb/pictures.c',
              'xcb/x11.c'
            ),
            'dependencies' : [xcb_dep, xcb_shm_dep],
            'link_with' : [vlc_xcb_events_lib]
        }
    endif
endif

egl_dep = dependency('egl', required: false)

if x11_dep.found() and egl_dep.found()
    vlc_modules += {
        'name' : 'egl_x11',
        'sources' : files('opengl/egl.c'),
        'c_args' : ['-DUSE_PLATFORM_X11=1'],
        'dependencies' : [egl_dep, x11_dep]
    }
endif

if x11_dep.found() and opengl_dep.found()
    vlc_modules += {
        'name' : 'glx',
        'sources' : files('glx.c'),
        'dependencies' : [x11_dep, opengl_dep]
    }
endif

if have_wayland
    wayland_include_dir = include_directories(meson.current_build_dir())

    viewporter_client_header = custom_target(
        'wayland-viewporter-client-header',
        output: 'viewporter-client-protocol.h',
        input: wayland_protocols_dir / 'stable/viewporter/viewporter.xml',
        command: [wayland_scanner, 'client-header', '@INPUT@', '@OUTPUT@'],
    )
    viewporter_protocol = custom_target(
        'wayland-viewporter-protocol',
        output: 'viewporter-protocol.c',
        input: wayland_protocols_dir / 'stable/viewporter/viewporter.xml',
        command: [wayland_scanner, 'private-code', '@INPUT@', '@OUTPUT@'],
    )

    vlc_modules += {
        'name' : 'wl_shm',
        'sources' : [
            files('wayland/registry.c', 'wayland/shm.c'),
            viewporter_client_header,
            viewporter_protocol,
        ],
        'include_directories' : [vlc_include_dirs, wayland_include_dir],
        'dependencies' : [wayland_deps],
    }

    xdg_shell_header = custom_target(
        'wayland-xdg-shell-client-header',
        output: 'xdg-shell-client-protocol.h',
        input: wayland_protocols_dir / 'stable/xdg-shell/xdg-shell.xml',
        command: [wayland_scanner, 'client-header', '@INPUT@', '@OUTPUT@'],
    )
    xdg_shell_protocol = custom_target(
        'wayland-xdg-shell-protocol',
        output: 'xdg-shell-protocol.c',
        input: wayland_protocols_dir / 'stable/xdg-shell/xdg-shell.xml',
        command: [wayland_scanner, 'private-code', '@INPUT@', '@OUTPUT@'],
    )
    xdg_decoration_header = custom_target(
        'wayland-xdg-decoration-client-header',
        output: 'xdg-decoration-client-protocol.h',
        input: wayland_protocols_dir / 'unstable/xdg-decoration/xdg-decoration-unstable-v1.xml',
        command: [wayland_scanner, 'client-header', '@INPUT@', '@OUTPUT@'],
    )
    xdg_decoration_protocol = custom_target(
        'wayland-xdg-decoration-protocol',
        output: 'xdg-decoration-protocol.c',
        input: wayland_protocols_dir / 'unstable/xdg-decoration/xdg-decoration-unstable-v1.xml',
        command: [wayland_scanner, 'private-code', '@INPUT@', '@OUTPUT@'],
    )

    xdg_c_args = []
    xdg_deps = [wayland_deps]
    if xkbcommon_dep.found()
        xdg_c_args += '-DHAVE_XKBCOMMON'
        xdg_deps += xkbcommon_dep
    endif

    xdg_shell_sources = [
        files(
            'wayland/output.c',
            'wayland/input.c',
            'xcb/xkb.c',
            'wayland/xdg-shell.c',
        ),
        xdg_shell_header,
        xdg_shell_protocol,
        xdg_decoration_header,
        xdg_decoration_protocol,
    ]

    vlc_modules += {
        'name' : 'wl_shell',
        'sources' : xdg_shell_sources,
        'include_directories' : [vlc_include_dirs, wayland_include_dir],
        'dependencies' : xdg_deps,
        'c_args' : xdg_c_args,
    }

    vlc_modules += {
        'name' : 'xdg_shell',
        'sources' : xdg_shell_sources,
        'include_directories' : [vlc_include_dirs, wayland_include_dir],
        'dependencies' : xdg_deps,
        'c_args' : xdg_c_args + ['-DXDG_SHELL'],
    }

    if egl_dep.found()
        vlc_modules += {
            'name' : 'egl_wl',
            'sources' : files('opengl/egl.c'),
            'c_args' : ['-DUSE_PLATFORM_WAYLAND=1'],
            'dependencies' : [egl_dep, wayland_deps],
        }

        gbm_dep = dependency('gbm', required: false)
        if gbm_dep.found() and drm_dep.found()
            linux_dmabuf_header = custom_target(
                'wayland-linux-dmabuf-client-header',
                output: 'linux-dmabuf-client-protocol.h',
                input: wayland_protocols_dir / 'unstable/linux-dmabuf/linux-dmabuf-unstable-v1.xml',
                command: [wayland_scanner, 'client-header', '@INPUT@', '@OUTPUT@'],
            )
            linux_dmabuf_protocol = custom_target(
                'wayland-linux-dmabuf-protocol',
                output: 'linux-dmabuf-protocol.c',
                input: wayland_protocols_dir / 'unstable/linux-dmabuf/linux-dmabuf-unstable-v1.xml',
                command: [wayland_scanner, 'private-code', '@INPUT@', '@OUTPUT@'],
            )

            vlc_modules += {
                'name' : 'egl_gbm_wl',
                'sources' : [
                    files('opengl/egl_gbm.c'),
                    linux_dmabuf_header,
                    linux_dmabuf_protocol,
                ],
                'include_directories' : [vlc_include_dirs, wayland_include_dir],
                'dependencies' : [egl_dep, wayland_deps, gbm_dep, drm_dep],
            }
        endif
    endif

    if vulkan_dep.found()
        vlc_modules += {
            'name' : 'vk_wl',
            'sources' : files('vulkan/platform.h', 'wayland/vulkan.c'),
            'dependencies' : [wayland_deps],
        }
    endif
endif
