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
<img src="images/idonotseeit.jpg" width=100 height=25>

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
```c
#include <yed/plugin.h>
```
So to interact with yed we need to include this header file, it gives us access to plugin functionality and all of the code used IN yed since this header file includes `internal.h`. <br />

```c
void recompile_init(int n_args, char **args);
```
Simple forward declaration
```c
int yed_plugin_boot(yed_plugin *self)
```

This line is the entry point of our program, yed plugins are just shared libraries but when yed calls the `.so`
file for access it needs to know where to start, so this is every plugins entry point,
think of it as `main` because this is how it's treated by everyone who has worked on yed plugins.

```c
YED_PLUG_VERSION_CHECK();
```

This is not actually a function, this is a macro, it checks that the plugin and yed versions match, otherwise it will not be loaded.

```c
yed_plugin_set_command(self, "recompile-init", recompile_init);
```

Now this is where it gets interesting, remember the forward declaration? We are adding that function as a command for yed, so when we load the plugin, in this case out init.c/so so it will always be loaded, we now get a `recompile-init` command. <br />
If you have not noticed by now, in yed functions in c files use snake_case and the editor commands use kebab-case.

```c
YEXE();
```
Finally we have `YEXE("[command]")`, this allows us to execute any command that's already available in yed in our c code, `YEXE`, `yed_plugin_set_command(...)` and event handlers are our bread and butter for plugin writing, while the last one is certainly not necessary, it is better to use it because it helps performance and keeps things consistent. 

## Event handlers
