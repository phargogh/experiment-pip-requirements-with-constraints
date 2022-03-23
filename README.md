# experiment-pip-requirements-with-constraints
Notes and experiments with pip requirements and constraints files.  RE: natcap/invest#880

## Problem

As noted in natcap/invest#880, pyinstaller on mac is getting the scipy 1.8.0
wheel, which looks like it's the first scipy wheel to include _both_ `arm` and
`x86_64` binaries of some of its DLLs.  Then, pyinstaller mistakenly includes
the `arm` binary instead of the `x86_64` one and we get a build failure on mac
([example](https://github.com/natcap/invest/runs/5111006848?check_suite_focus=true)).

The [recommended workaround](https://github.com/scipy/scipy/issues/15552#issuecomment-1048303347)
is to use the `x86_64` wheel instead of the `universal2` wheel.  That's all
well and good, but how do we do this in a way that works for InVEST's CI?

## Hypothesis

Pip has long supported [constraint files](https://pip.pypa.io/en/stable/user_guide/#constraints-files)
as a complement to [requirements files](https://pip.pypa.io/en/stable/reference/requirements-file-format/),
however the documentation is quite light.  Because the intent of the
constraints file is to place limitations on the packages being installed, I
suspect we can use this to force pip to install the `scipy` build we're looking for.

## Setup and Testing

```
$ mamba create -p ./env -c conda-forge -y python=3.9
$ conda activate ./env
$ pip install -r requirements.txt -c constraints.txt
```

## Notes from experimentation

* On my M1 mac, if I `pip install scipy --platform macosx_10_9_x86_64  --python-version="3.9" --only-binary scipy --no-deps`, I get the correct wheel.
* Looks like we can specify which wheel to use!  We can provide the URL to the pypi-hosted wheel to use for scipy in constraints.txt.

## Conclusion

Providing a URL to the specific wheel we want is a valid constraint for the requirement.
