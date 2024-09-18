project('avro-cpp', 'cpp', version: '1.0', 
  default_options: [
    'cpp_std=c++17',
    'warning_level=3',
    'werror=true'
  ])

# Compiler and platform-specific settings
if host_machine.system() == 'windows'
  add_project_arguments(
    '/EHa',
    '-DNOMINMAX',
    '-DBOOST_REGEX_DYN_LINK',
    '-DBOOST_FILESYSTEM_DYN_LINK',
    '-DBOOST_SYSTEM_DYN_LINK',
    '-DBOOST_IOSTREAMS_DYN_LINK',
    '-DBOOST_PROGRAM_OPTIONS_DYN_LINK',
    '-DBOOST_ALL_NO_LIB',
    language: 'cpp'
  )
elif host_machine.system() == 'darwin'
  # macOS specific options
  add_project_arguments('-DMACOSX_RPATH', language: 'cpp')
else
  # General GCC flags
  add_project_arguments(
    '-Wduplicated-cond', '-Wduplicated-branches', '-Wlogical-op',
    '-Wuseless-cast', '-Wconversion', '-pedantic',
    language: 'cpp'
  )
endif

# Boost dependencies
boost_dep = dependency('boost', modules: ['filesystem', 'iostreams', 'program_options', 'regex', 'system'])

# Snappy dependency
snappy_dep = dependency('snappy', required: false)
if snappy_dep.found()
  add_project_arguments('-DSNAPPY_CODEC_AVAILABLE', language: 'cpp')
endif

# fmt dependency
fmt_dep = dependency('fmt', required: true)

# Versioning from file
fs = import('fs')
version_content = fs.read('../../share/VERSION.txt').strip()
avro_version = version_content.strip().split('.')
avro_version_major = avro_version[0]
avro_version_minor = avro_version[1]
avro_version_patch = avro_version[2]

add_project_arguments('-DAVRO_VERSION="'+version_content +'"', language : 'cpp')

# Common source files
avro_sources = [
  'impl/Compiler.cc', 'impl/Node.cc', 'impl/LogicalType.cc',
  'impl/NodeImpl.cc', 'impl/ResolverSchema.cc', 'impl/Schema.cc',
  'impl/Types.cc', 'impl/ValidSchema.cc', 'impl/Zigzag.cc',
  'impl/BinaryEncoder.cc', 'impl/BinaryDecoder.cc', 'impl/Stream.cc',
  'impl/FileStream.cc', 'impl/Generic.cc', 'impl/GenericDatum.cc',
  'impl/DataFile.cc', 'impl/parsing/Symbol.cc', 'impl/parsing/ValidatingCodec.cc',
  'impl/parsing/JsonCodec.cc', 'impl/parsing/ResolvingDecoder.cc',
  'impl/json/JsonIO.cc', 'impl/json/JsonDom.cc', 'impl/Resolver.cc', 
  'impl/Validator.cc', 'impl/CustomAttributes.cc'
]

# Create shared and static libraries
libavro = library('avrocpp', avro_sources, 
  include_directories: include_directories('include/avro'),
  dependencies: [boost_dep, snappy_dep, fmt_dep],
  version: '@0@.@1@.@2@'.format(avro_version_major, avro_version_minor, avro_version_patch.split('-')[0]),
  install: true
)

# AvroGenCpp executable
avrogencpp_exe = executable('avrogencpp', 'impl/avrogencpp.cc',
  link_with: libavro,
  include_directories: include_directories('include/avro'),
  dependencies: [boost_dep, snappy_dep, fmt_dep],
  install: true
)

# Precompile test executable
precompile_exe = executable('precompile', 'test/precompile.cc',
  link_with: libavro,
  include_directories: include_directories('include/avro'),
  dependencies: [boost_dep, snappy_dep, fmt_dep]
)

if get_option('tests')

  jsonschema_list = [
    ['empty_record','empty'],
    ['bigrecord','testgen'],
    ['bigrecord_r','testgen_r'],
    ['bigrecord2','testgen2'],
    ['tweet','testgen3'],
    ['union_array_union','uau'],
    ['union_map_union','umu'],
    ['union_conflict','uc'],
    ['union_empty_record','uer'],
    ['recursive','rec'],
    ['reuse','ru'],
    ['circulardep','cd'],
    ['tree1','tr1'],
    ['tree2','tr2'],
    ['crossref','cr'],
    ['primitivetypes','pt'],
    ['cpp_reserved_words','cppres'],
    ['cpp_reserved_words_union_typedef','cppres_union'],
    ['big_union','big_union'],
    ['union_redundant_types','redundant_types']
  ]

  schema_headers = []
  foreach jsonschema : jsonschema_list
   schema_headers += custom_target(
      jsonschema[0],
      input : 'jsonschemas/'+jsonschema[0],
  		output : jsonschema[0]+'.hh',
      command : [avrogencpp_exe,'-p','-','-o','@OUTPUT@','-i', '@INPUT0@', '-n', jsonschema[1]]
  )
  endforeach

  # Add tests
  test_list = [
    'buffertest', 'unittest', 'SchemaTests', 'LargeSchemaTests',
    'CodecTests', 'StreamTests', 'SpecificTests', 'DataFileTests',
    'JsonTests', 'AvrogencppTests', 'CompilerTests', 'AvrogencppTestReservedWords',
    'CommonsSchemasTests'
  ]

  foreach test_name : test_list
    test_exe = executable(test_name, 
      sources: schema_headers + ['test/@0@.cc'.format(test_name)],
      include_directories: include_directories('include/avro'),
      link_with: libavro,
      dependencies: [boost_dep, snappy_dep,fmt_dep]
    )
    test(test_name, test_exe)
  endforeach

endif


avro_dep = declare_dependency(include_directories: include_directories('include'), link_with: libavro)
