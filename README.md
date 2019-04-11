# nix-mode and nix-shell - Major mode for nix expressions and integration of Emacs to nix-shell with nix-shell

*Author:* Mario Rodas <marsam@users.noreply.github.com><br>
*Version:* 0.1<br>

A major mode for [Nix][] expressions.

## Usage of nix-shell (this section added by @analyticd)
Mario has solved the problem of getting Emacs to use nix-shell
executables in Emacs without having to start Emacs from within a
nix-shell! This ends much longstanding grief for many Nix users,
myself included. Thanks Mario!

(use-package nix-mode
  :load-path (lambda () (expand-file-name "github.com/analyticd/nix-mode" ghq-root))) ; just the path to the repo will do

;; M-x nix-shell-active will look for your default.nix or shell.nix
;; and make sure to find the proper nix-shell-based executables. Works
;; great for Python for instance. No more need for direnv integration
;; with Emacs (which is very slow).
(use-package nix-shell
   :load-path (lambda () (expand-file-name "github.com/analyticd/nix-mode" ghq-root))) 
   
Here is an example shell.nix for Python use (in my case using a tarball from the
stable channel to ensure that binaries and not source are used - no
compiling please, this is Nix after all - I'm looking at you pandas on
unstable channel):

{
  pkgs ? import (fetchTarball {
    url = https://github.com/NixOS/nixpkgs-channels/archive/nixos-18.09.tar.gz;
    # sha256 = "0xvj0z928mzcs56hindxw608a6jm1fvsi2ap5r7m4a0plnrcwx90";
  }) {}
}:
pkgs.mkShell {
  buildInputs = with pkgs; [
    python3Full
    python3Packages.pip
    python3Packages.numpy
    python3Packages.pandas
    python3Packages.matplotlib
    python3Packages.pyyaml
    python3Packages.scipy
    python3Packages.pillow
    python3Packages.pymongo
  ];
  shellHook = ''
  # echo 'Entering Python Project Environment'
  # set -v

  # If you want to use pip:
  # extra packages can be installed here
  unset SOURCE_DATE_EPOCH
  export PIP_PREFIX="$(pwd)/_build/pip_packages"
  python_path=(
    "$PIP_PREFIX/lib/python3/site-packages"
    "$PYTHONPATH"
  )
  # use double single quotes to escape bash quoting
  IFS=: eval 'python_path="''${python_path[*]}"'
  export PYTHONPATH="$python_path"
  # export MPLBACKEND='Qt4Agg'  # Put this in ~/.matplotlib/matplotlibrc instead. On OS X, use 'backend: MacOS'

  # set +v
  '';
}

Here is an example shell.nix for multiple language use (Python,
Haskell) in, say, an org-babel environment for executing org source
blocks (placed in your org-directory):

{
  pkgs ? import (fetchTarball {
    url = https://github.com/NixOS/nixpkgs-channels/archive/nixos-18.09.tar.gz;
    # sha256 = "0xvj0z928mzcs56hindxw608a6jm1fvsi2ap5r7m4a0plnrcwx90";
  }) {}
}:
pkgs.mkShell {
  buildInputs = with pkgs; [
    python3Full
    python3Packages.pip
    python3Packages.numpy
    python3Packages.pandas
    python3Packages.matplotlib
    python3Packages.pyyaml
    python3Packages.scipy
    # python3Packages.pillow
    # python3Packages.pymongo
    (ghc.withPackages (p: [
      p.lens p.aeson p.lens-aeson p.text p.vector p.hmatrix
    ]))
  ];
  shellHook = ''
  # eval "$(direnv hook zsh)"  # Put this in my .zshrc
  # direnv allow .

  # echo 'Entering Python Project Environment'
  # set -v

  # extra packages can be installed here
  unset SOURCE_DATE_EPOCH
  export PIP_PREFIX="$(pwd)/../../../src/pip_packages"
  python_path=(
    "$PIP_PREFIX/lib/python3/site-packages"
    "$PYTHONPATH"
  )
  # use double single quotes to escape bash quoting
  IFS=: eval 'python_path="''${python_path[*]}"'
  export PYTHONPATH="$python_path"

  set +v
  '';
}

### TODO

+ [ ] Indentation with `smie`.  See: https://github.com/NixOS/nix/blob/master/doc/manual/nix-lang-ref.xml

[Nix]: https://nixos.org/nix/


---
Converted from `nix-mode.el` by [*el2markdown*](https://github.com/Lindydancer/el2markdown).

