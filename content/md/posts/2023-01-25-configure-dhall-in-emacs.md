{:layout :post, :title "Configuring Dhall with Emacs", :date "2023-01-25" :draft? false :tags ["linux" "dhall" "emacs"]}

I use Dhall with archlinux, and emacs, but don't use `use-package`. The documentation at https://docs.dhall-lang.org/howtos/Text-Editor-Configuration.html#EMACS assumes you do use this, but I use `prelude` to simplify a bunch of my emacs setup, and am not interested in another tool that does similar setup work.

To get Dhall (and in particular, the lsp server) setup, the following steps worked for me.

1. Install dhall, dhall-json, and dhall-lsp-server:
``` shell
pacman -Syu dhall dhall-json dhall-lsp-server
```
2. Ensure they're on the path, and smoke-test dhall-json:
``` shell
dhall-to-json <<< '{ foo = [1, 2, 3], bar = True }'
```
3. Install `dhall-mode` in emacs

``` text
M-x package-install
Type dhall-mode
Then enter to install
```

4. Configure emacs to use dhall, and the lsp server
``` emacs-lisp
;; Setup Dhall
(require 'dhall-mode)
(setq
 ;; comment the next line to use unicode syntax
 dhall-format-arguments (\` ("--ascii"))

 ;; header-line is obsoleted by lsp-mode
 dhall-use-header-line nil)

(add-hook 'dhall-mode-hook 'lsp)
```
5. Evaluate the above in emacs, and then open a `.dhall` file. lsp should start up, and be ready to go.

You can test the integration with the following configuration file

``` dhall
let user = "Dhall"

let welcome = \(name : Text) -> "Welcome ${name}"

in  welcome user ++ 42
```

It should warn you on the last line with `Error: (++) only works on (Text)`
