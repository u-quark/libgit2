How to compile with clang similar to libgit2 CI
===============================================

Local diff:

```
diff --git a/cmake/EnableWarnings.cmake b/cmake/EnableWarnings.cmake
index 0700b521b..7672312c7 100644
--- a/cmake/EnableWarnings.cmake
+++ b/cmake/EnableWarnings.cmake
@@ -1,5 +1,6 @@
 macro(ENABLE_WARNINGS flag)
        add_c_flag_if_supported(-W${flag})
+       add_c_flag_if_supported(-Wno-unused-command-line-argument)
 endmacro()
 
 macro(DISABLE_WARNINGS flag)
```

Compilation flags for nix derivation in gg:

```
version = "test";
cmakeFlags = old.cmakeFlags ++ [ "-DCMAKE_C_COMPILER=clang" "-DENABLE_WERROR=ON" "-DBUILD_EXAMPLES=ON" "-DBUILD_FUZZERS=ON" "-DUSE_STANDALONE_FUZZERS=ON" "-G" "Ninja" "-DUSE_SHA1=HTTPS" "-DREGEX_BACKEND=pcre" "-DDEPRECATE_HARD=ON" "-DUSE_LEAK_CHECKER=valgrind" "-DUSE_GSSAPI=OFF" "-DUSE_SSH=exec" ];
nativeBuildInputs = old.nativeBuildInputs ++ [ pkgs.binutils pkgs.ninja pkgs.clang pkgs.valgrind ];
postBuild = ''
  ./libgit2_tests
...
```

Patches I had to do to compile without errors:

```
diff --git a/src/util/net.c b/src/util/net.c
index 447456451..b11d15588 100644
--- a/src/util/net.c
+++ b/src/util/net.c
@@ -460,7 +460,7 @@ done:
 int git_net_url_parse(git_net_url *url, const char *given)
 {
        git_net_url_parser parser = GIT_NET_URL_PARSER_INIT;
-       const char *c, *authority, *path;
+       const char *c, *authority, *path = NULL;
        size_t authority_len = 0, path_len = 0;
        int error = 0;
 
@@ -657,7 +657,7 @@ static bool has_at(const char *str)
 int git_net_url_parse_scp(git_net_url *url, const char *given)
 {
        const char *default_port = default_port_for_scheme("ssh");
-       const char *c, *user, *host, *port = NULL, *path = NULL;
+       const char *c, *user = NULL, *host, *port = NULL, *path = NULL;
        size_t user_len = 0, host_len = 0, port_len = 0;
        unsigned short bracket = 0;
```
