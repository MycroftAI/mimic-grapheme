project('mimic-grapheme', 'c', default_options : ['c_std=c99'], version: '0')

build_plugin = get_option('plugin')

sources = [
  'src/cmu_grapheme_lang/cmu_grapheme_lang.c',
  'src/cmu_grapheme_lang/cmu_grapheme_phoneset.c',
  'src/cmu_grapheme_lang/cmu_grapheme_phrasing_cart.c',
  'src/cmu_grapheme_lang/cmu_grapheme_phrasing_cart.h',
  'src/cmu_grapheme_lex/cmu_grapheme_lex.c',
  'src/cmu_grapheme_lex/grapheme_unitran_tables.c'
]

includes = [include_directories('include')]

# Core Dependency:
mimiccore = dependency('ttsmimic_core', required: false)
if mimiccore.found()
mimic_plugin_dir = mimiccore.get_pkgconfig_variable('mimic_plugin_dir')
else
  # We compile it:
  mimic_core_sub = subproject('mimic-core')
  mimiccore = mimic_core_sub.get_variable('ttsmimiccore_dep')
  mimic_plugin_dir = mimic_core_sub.get_variable('mimic_plugin_dir')
endif

# If mimic-core does not have plugin support, disable the plugin
if mimic_plugin_dir == ''
    build_plugin = false
endif
message('Build mimic-grapheme plugin: @0@'.format(build_plugin))


grapheme_header_subdir = join_paths('ttsmimic', 'lang', 'grapheme')

conf_data = configuration_data()
conf_data.set('ENABLE_LANG_GRAPHEME_BUILTIN', not build_plugin)

mimic_grapheme_config_h = configure_file(output : 'mimic_grapheme_config.h',
                                         configuration : conf_data, install: true,
                                         install_dir : join_paths(get_option('includedir'), grapheme_header_subdir))

sources += [mimic_grapheme_config_h]

ttsmimic_grapheme_lib = library('ttsmimic_grapheme', sources,
                                include_directories : includes,
                                dependencies: mimiccore,
                                install: true)

ttsmimic_grapheme_dep = declare_dependency(link_with : ttsmimic_grapheme_lib, 
                                              include_directories : includes)


if build_plugin
  ttsmimic_grapheme_plugin_lib = library(
    'ttsmimic_grapheme_plugin',
    ['src/mimic_grapheme_plugin.c'],
    include_directories : includes,
    dependencies: [mimiccore, ttsmimic_grapheme_dep],
    install_dir: mimic_plugin_dir,
    install: true)

  ttsmimic_grapheme_plugin_dep = declare_dependency(
    link_with : ttsmimic_grapheme_plugin_lib, 
    include_directories : includes)
endif



############ Headers #################

ttsmimic_headers = ['include/cmu_grapheme_lang.h', 'include/cmu_grapheme_lex.h']

install_headers(ttsmimic_headers, subdir : grapheme_header_subdir)

########### pkg-config #############

pkg = import('pkgconfig')

pkg.generate(libraries : ttsmimic_grapheme_lib,
             name : 'ttsmimic_grapheme',
             subdirs: grapheme_header_subdir,
             requires: 'ttsmimic_core',
             description: 'Grapheme support for text to speech synthesis in mimic.',
             version: meson.project_version())

