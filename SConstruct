import sys
import re
import glob
import excons
from excons.tools import gl

env = excons.MakeBaseEnv()

if sys.platform != "win32":
   env.Append(CPPFLAGS=" -Wno-unused-parameter")

defs = []
incs = ["include", "deps"]
srcs = ["src/context.c",
        "src/init.c",
        "src/input.c",
        "src/monitor.c",
        "src/vulkan.c",
        "src/window.c"]
libs = []
cust = [gl.Require]

if sys.platform == "win32":
   defs.extend(["_GLFW_WIN32",
                "_CRT_SECURE_NO_WARNINGS"])

   srcs.extend(["src/win32_init.c",
                "src/win32_joystick.c",
                "src/win32_monitor.c",
                "src/win32_time.c",
                "src/win32_tls.c",
                "src/win32_window.c",
                "src/wgl_context.c",
                "src/egl_context.c"])
  
elif sys.platform == "darwin":
   def AddFrameworks(env):
      env.Append(LINKFLAGS=["-framework", "Cocoa",
                            "-framework", "IOKit",
                            "-framework", "CoreFoundation",
                            "-framework", "CoreVideo"])
  
   defs.extend(["_GLFW_COCOA",
                "_GLFW_USE_CHDIR",
                "_GLFW_USE_MENUBAR"])
  
   srcs.extend(["src/cocoa_init.m",
                "src/cocoa_joystick.m",
                "src/cocoa_monitor.m",
                "src/cocoa_window.m",
                "src/cocoa_time.c",
                "src/posix_tls.c",
                "src/nsgl_context.m",
                "src/egl_context.c"])
  
   cust.append(AddFrameworks)
  
else:
   defs.extend(["_GLFW_X11",
                "_GLFW_GLX",
                "_GLFW_HAS_DLOPEN",
                "_GLFW_HAS_GLXGETPROCADDRESS"])
  
   srcs.extend(["src/x11_init.c",
                "src/x11_monitor.c",
                "src/x11_window.c",
                "src/xkb_unicode.c",
                "src/linux_joystick.c",
                "src/posix_time.c",
                "src/posix_tls.c",
                "src/glx_context.c",
                "src/egl_context.c"])
  
   libs.extend(["X11", "Xcursor", "Xrandr", "Xinerama"])
   cust.append(lambda x: x.Append(CCFLAGS="-fvisibility=hidden"))

InstallHeaders = env.Install("%s/include/GLFW" % excons.OutputBaseDirectory(), glob.glob("include/GLFW/*.h"))

prjs = [
   {  "name": "glfw3_s",
      "type": "staticlib",
      "desc": "GLFW static library",
      "defs": defs,
      "incdirs": incs,
      "srcs": srcs
   },
   {  "name": ("glfw3" if sys.platform == "win32" else "glfw"),
      "alias": "glfw3",
      "type": "sharedlib",
      "desc": "GLFW shared library",
      "version": "3.3.0",
      "soname": "libglfw.so.3",
      "install_name": "libglfw.3.dylib",
      "defs": defs + ["_GLFW_BUILD_DLL"],
      "incdirs": incs,
      "srcs": srcs,
      "libs": libs,
      "custom": cust
   }
]

tgts = excons.DeclareTargets(env, prjs)

env.Depends("glfw3", InstallHeaders)
env.Depends("glfw3_s", InstallHeaders)

def RequireGLFW(static=True):
   def _RealRequire(env):
      if not static:
         env.Append(CPPDEFINES=["GLFW_DLL"])
         
         if sys.platform == "win32":
            env.Append(LIBS=["glfw3"])
         else:
            env.Append(LIBS=["glfw"])

      else:
         libs = ["glfw3_s"]
         if sys.platform == "darwin":
            env.Append(LINKFLAGS=["-framework", "Cocoa",
                                  "-framework", "IOKit",
                                  "-framework", "CoreFoundation",
                                  "-framework", "CoreVideo"])
         elif sys.platform != "win32":
            libs.extend(["X11", "Xcursor", "Xrandr", "Xinerama", "dl"])

         env.Append(LIBS=libs)

      gl.Require(env)
   
   return _RealRequire

Export("RequireGLFW")
