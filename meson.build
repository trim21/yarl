project(
    'yarl',
    'c',
    default_options: [
        'buildtype=release',
        'debug=false',
    ],
    meson_version: '>= 1.3.0',
)

py_inter = import('python').find_installation()

PY_IMPL = run_command(
    py_inter,
    '-c', 'import sys; print(sys.implementation.name)',
    check: true,
).stdout().strip()

pure_py = (get_option('pure-python').enabled()) or (PY_IMPL != 'cpython')

py = import('python').find_installation(pure: pure_py)

py.install_sources(
    [
        'yarl/__init__.py',
        'yarl/py.typed',
        'yarl/_parse.py',
        'yarl/_path.py',
        'yarl/_query.py',
        'yarl/_quoters.py',
        'yarl/_quoting.py',
        'yarl/_quoting_py.py',
        'yarl/_url.py',
    ],
    subdir: 'yarl',
)

if not pure_py
    cython = find_program('cython')

    cython_gen = generator(
        cython,
        output: ['@BASENAME@.c'],
        arguments: ['@INPUT@', '--output-file', '@OUTPUT0@'],
    )

    quoting_c = cython_gen.process(
        'yarl/_quoting_c.pyx',
        preserve_path_from: meson.current_source_dir(),
    )

    out = py.extension_module(
        '_quoting_c',
        quoting_c,
        subdir: 'yarl',
        install: true,
        dependencies: py.dependency(),
    )

    # support for in-tree build
    # # will target will copy binary extension back to source directory
    custom_target(
        'copy extension back to file tree',
        input: out,
        output: 'copy',
        depends: out,
        command: [
            'cp',
            out.full_path(),
            join_paths(meson.project_source_root(), 'yarl/'),
        ],
        build_by_default: false,
    )
endif
