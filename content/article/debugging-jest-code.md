+++
Categories = ["Code"]
Description = "Visual Studio Code comes with great features, one of which is debuging. Let's set it up for debugging tests implemented with Facebook's Jest framework."
Tags = ["JavaScript"]
date = "2016-03-23T22:26:58+01:00"
title = "Debugging Jest in Visual Studio Code"
draft = false
+++
One of the reason I decided to start using
[Visual Studio Code](https://code.visualstudio.com) over [Atom](https://atom.io)
is due to the built-in debugger. Others are the integrated Git view, and that I
find it faster than Atom (miss being able to have multiple projects open in the
same window though).

In one of my pet-projects I started to use [Jest](https://github.com/facebook/jest)
from Facebook as my Unit Testing framework of choice. And getting support for
debugging my tests was only a matter of updating the `.vscode/launch.json` file
with this:

    {
        "version": "0.2.0",
        "configurations": [
            {
                "name": "Tests",
                "type": "node",
                "request": "launch",
                "program": "${workspaceRoot}/node_modules/jest-cli/bin/jest.js",
                "stopOnEntry": false,
                "args": ["--runInBand"],
                "cwd": "${workspaceRoot}",
                "preLaunchTask": null,
                "runtimeExecutable": null,
                "runtimeArgs": [
                    "--nolazy"
                ],
                "env": {
                    "NODE_ENV": "development"
                },
                "externalConsole": false,
                "sourceMaps": false,
                "outDir": null
            }
        ]
    }

The significant line here, is the use of argument `--runInBand` which will
cause Jest to run all tests in a single sequence in the current process
(instead of spinning up multiple processes to run tests in parallel). You want
to use this setting only for debugging, not for normal test runs.

If you have not yet tried Code, I recommend you give it a try, it's awesome.

Happy hacking!
