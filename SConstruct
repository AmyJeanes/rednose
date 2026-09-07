import os
import platform
import subprocess
import sys
import sysconfig
import numpy as np
import eigen

WINDOWS = platform.system() == "Windows"
arch = subprocess.check_output(["uname", "-m"], encoding='utf8').rstrip()

common = ''

python_path = sysconfig.get_paths()['include']
cpppath = [
  '#',
  '#rednose',
  '#rednose/examples/generated',
  '/usr/lib/include',
  python_path,
  np.get_include(),
  eigen.INCLUDE_DIR,
]

env = Environment(
  ENV=os.environ,
  CCFLAGS=[
    "-g",
    "-fPIC",
    "-O2",
    "-Werror=implicit-function-declaration",
    "-Werror=incompatible-pointer-types",
    "-Werror=int-conversion",
    "-Werror=return-type",
    "-Werror=format-extra-args",
    "-Wshadow",
  ],
  LIBPATH=["#rednose/examples/generated"],
  CFLAGS="-std=gnu11",
  CXXFLAGS="-std=c++1z",
  CPPPATH=cpppath,
  REDNOSE_ROOT=Dir("#").abspath,
  tools=["mingw" if WINDOWS else "default", "cython", "rednose_filter"],  # the default tool picks MSVC on Windows
  # the mingw tool assumes gcc and drops the lib prefix ekf_load expects; static libc++ so the DLLs load outside the MSYS2 shell
  **({"CC": "clang", "CXX": "clang++", "SHLIBPREFIX": "lib", "LINKFLAGS": ["-static"]} if WINDOWS else {}),
)

# Cython build enviroment
envCython = env.Clone()
envCython["CCFLAGS"] += ["-Wno-#warnings", "-Wno-cpp", "-Wno-shadow", "-Wno-deprecated-declarations"]

envCython["LIBS"] = []
if platform.system() == "Darwin":
  envCython["LINKFLAGS"] = ["-bundle", "-undefined", "dynamic_lookup"]
elif WINDOWS:
  envCython["LINKFLAGS"] = ["-shared", "-static"]
  envCython["LIBPATH"] = [os.path.join(sys.base_prefix, "libs")]
  envCython["LIBS"] = [f"python{sys.version_info.major}{sys.version_info.minor}"]
  # extension modules are .pyd on Windows; the SConscript names them .so
  def _pyd_emitter(target, source, env):
    return [env.File(str(t)[:-3] + ".pyd") if str(t).endswith(".so") else t for t in target], source
  envCython.Append(PROGEMITTER=[_pyd_emitter])
elif arch == "aarch64":
  envCython["LINKFLAGS"] = ["-shared"]
  envCython["LIBS"] = [os.path.basename(python_path)]
else:
  envCython["LINKFLAGS"] = ["-pthread", "-shared"]

Export('env', 'envCython', 'common')

SConscript(['#rednose/SConscript'])
SConscript(['#examples/SConscript'])
