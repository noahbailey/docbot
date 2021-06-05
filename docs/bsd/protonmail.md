# Protonmail Bridge on OpenBSD

## Prereqs

Install golang and GNU make: 

    pkg_add go gmake password-store

Make sure the `go` packages are in the PATH: 

    export PATH="${PATH}:$(go env GOPATH)/bin"

## Password manager (Pass)

Create a GPG key for Pass to use (Has no password)

    gpg --batch --passphrase '' --quick-gen-key 'ProtonMail Bridge' default default never

Initialize the password store (if not already done): 

    pass init <GPG_KEY_ID>

    pass git init

## Installing

Clone the source tree: 

    git clone git@github.com:ProtonMail/proton-bridge.git

Copy one file (Allows the TLS library to be installed)

    cp internal/config/tls/cert_store_linux.go internal/config/tls/cert_store_openbsd.go

Compile the software: 

    TARGET_OS="linux" gmake build-nogui


