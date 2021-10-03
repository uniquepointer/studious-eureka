# Preliminary dev manual for yed

## wut

## Baby steps

If after a fresh install and letting `yed` setup things for you, you take a peek to your `~/.config/yed` you will see the `init.c`,
despite being your init file, this also has regular patterns of `yed` plugins.
```c
#include <yed/plugin.h>

void recompile_init(int n_args, char **args);

int yed_plugin_boot(yed_plugin *self) {
    char *path;

    YED_PLUG_VERSION_CHECK();

    yed_plugin_set_command(self, "recompile-init", recompile_init);

    YEXE("plugin-load", "yedrc");

    path = get_config_item_path("yedrc");

    YEXE("yedrc-load", path);

    free(path);

    return 0;
}

void recompile_init(int n_args, char **args) {
    const char *config_path;
    char        buff[4096];

    config_path = get_config_path();

    snprintf(buff, sizeof(buff),
             "gcc -o %s/init.so %s/init.c $(yed --print-cflags --print-ldflags) && echo success",
             config_path, config_path);

    YEXE("sh", buff);
}

```

Now while in your config folder the `init.c` will have comments, we will pretend we did not see them.
<img src="images/idonotseeit.jpg">

What I do want you to see is the
```c
#include <yed/plugin.h>

void recompile_init(int n_args, char **args);

int yed_plugin_boot(yed_plugin *self){}
// . . .
YED_PLUG_VERSION_CHECK();

yed_plugin_set_command(self, "recompile-init", recompile_init);
// . . .
YEXE(. . .);
```

Let's start at the beginning.<br />
```
#include <yed/plugin.h>
```
So to interact with yed we need to include this header file, it gives us access to plugin functionality and all of the code used IN yed since this header file includes `internal.h`. <br />

```
void recompile_init(int n_args, char **args);
```
Simple forward declaration
```
int yed_plugin_boot(yed_plugin *self)
```

This line is the entry point of our program, yed plugins are just shared libraries but when yed calls the `.so`
file for access it needs to know where to start, so this is every plugins entry point,
think of it as `main` because this is how it's treated by everyone who has worked on yed plugins.

```
YED_PLUG_VERSION_CHECK();
```

This is not actually a function, this is a macro, it checks that the plugin and yed versions match, otherwise it will not be loaded.

```
void recompile_init(int n_args, char **args);
```

Now this is where it gets interesting
