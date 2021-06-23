# Rfx : ReFactor Elixir

Rfx provides a catalog of automated refactoring operations for Elixir source
code.  

**NOTE** at the moment this code is under heavy development!  It will change
rapidly, and there will be bugs!

To get started with this pre-release code, clone the repo, then run `> mix test
--exclude pending`.

## Ops Modules

See `Rfx.Catalog.OpsCat` for a complete list of operations.

Module Operations:

- [ ] rename module (`Rfx.Ops.Module.RenameModule`)
- [ ] rename module attribute
- [ ] extract function
- [ ] inline function

Function Operations:

- [ ] rename function
- [ ] rename function parameter
- [ ] rename variable
- [ ] extract variable
- [ ] inline variable

Filesystem Operations:

- [ ] move directory (`Rfx.Ops.Filesys.MvDir`)
- [ ] move file (`Rfx.Ops.Filesys.MvFile`)

Surface Operations:

- [ ] rename component
- [ ] rename property
- [ ] rename named-slot

PhxGen Operations:

- [ ] add route
- [ ] add controller

PhxGenAuth Operations:

- [ ] add controller

Project Operations:

- [ ] add dependency 
- [ ] increment version

Credo Operations:

- [x] multi-alias (`Rfx.Ops.Credo.MultiAlias`)

Prototype Operations:

- [x] comment add (`Rfx.Ops.Proto.CommentAdd`)
- [x] comment del (`Rfx.Ops.Proto.CommentDel`)
- [x] no-op (`Rfx.Ops.Proto.NoOp`)

Rfx Operations depend on the excellent
[Sourceror](http://github.com/doorgan/sourceror) written by
[@doorgan](http://github.com/doorgan).

## Change Sets

Each operation returns a *change set* (`Rfx.Change.Set`) with a list of of
*change requests* (`Rfx.Change.Request`).  The *change set* is a data structure
that describes all the refactoring changes to be made for an operation.

A *change request* struct has elements for *text edits* and *file actions*
(create, rename, delete).

## Helper Functions

Rfx provides a catalog of conversion functions to annotate changesets:

```elixir
Rfx.Change.Set.convert(changeset, :to_string)    #> Returns the modified source code
Rfx.Change.Set.convert(changeset, :to_patchfile) #> Returns a unix-standard patchfile
Rfx.Change.Set.convert(changeset, :to_lsp)       #> Returns a data structure for LSP
Rfx.Change.Set.convert(changeset, to_pr)         #> Returns a pull-request data structure
```

Rfx provides an `apply` function that applies the change requests to the filesystem.

```elixir
Rfx.Change.Set.apply!(changeset)                 #> Applies the changereqs to the filesystem
```

## Code Organization

Each operation is coded in a standalone module that implements the `Rfx.Ops`
behavior.  A refactoring operation may be applied to different scopes:

- Scope1: a code string
- Scope2: a single file
- Scope3: an entire project
- Scope4: an umbrella sub-application
- Scope5: a temp file

Each scope will generate a different set of change requests.  Consider for
example the operation `Rfx.Ops.Module.RenameModule`.

| Rename_Module    | # of Change Requests            | Text Edits      | File Actions           |
|------------------|---------------------------------|-----------------|------------------------|
| Scope1 `code`    | 1                               | Edit src & docs | NA                     |
| Scope2 `file`    | 1                               | Edit src & docs | Rename Src file        |
| Scope3 `project` | 1 for each related project file | Edit src & docs | Rename Src & Test file |
| Scope4 `subapp`  | 1 for each related subapp file  | Edit src & docs | Rename Src & Test file |
| Scope5 `tmpfile` | 1                               | Edit src & docs | NA                     |

Here's a pseudo-code example:

```elixir
alias Rfx.Ops.Module.RenameModule
alias Rfx.Change

# return edited source code
RenameModule.cl_code(input_code) 
| > Change.Cast.to_string()
#> {:ok, edited_source_code_string}

# write changes to file system
RenameModule.cl_file(input_file_name, new_name: "MyNewName") 
|> Change.Cast.apply!()
#> :ok  

# return a unix patchfile string
RenameModule.cl_project(project_dir, old_module: "OldModule", new_module: "NewModule") 
|> Change.Cast.to_patch()
#> {:ok, list of patchfile_strings}
```

## Clients 

Rfx Operations are meant to be embedded into editors, tools and end-user
applications:

- Tests, Elixir Scripts and LiveNotebooks
- Generators (eg phx.gen, phx.gen.auth)
- CLI (see the experimental [rfx_cli](https://github.com/andyl/rfx_cli))
- Editor Plugins (see the experimental [rfx_nvim](https://github.com/andyl/rfx_nvim))
- Mix tasks 
- ElixirLs
- Credo

## Installation

```elixir
def deps do
  [
    {:rfx, github: "andyl/rfx"}
  ]
end
```

