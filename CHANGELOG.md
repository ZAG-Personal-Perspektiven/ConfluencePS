# Change Log

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](http://keepachangelog.com/),
and this project adheres to [Semantic Versioning](http://semver.org/).

## 2.3 2018-03-22

FEATURES:

- Added custom object type for Attachments: `ConfluencePS.Attachment` (#123, [@JohnAdders][])
- Added `Add-Attachment`: upload a file to a page (#123, [@JohnAdders][])
- Added `Get-Attachment`: list all attachments of a page (#123, [@JohnAdders][])
- Added `Get-AttachmentFile`: download an attachment to the local disc (#123, [@JohnAdders][])
- Added `Remove-Attachment`: remove an attachment from a page (#123, [@JohnAdders][])
- Added `Set-Attachment`: update an attachment of a page (#123, [@JohnAdders][])
- Made `Invoke-Method` public (#130, [@lipkau][])
- Added `-InFile` to `Invoke-Method` for uploading of files with `form-data` (#130, [@lipkau][])

IMPROVEMENTS:

- Moved Online Help of cmdlets to the homepage (#130, [@lipkau][])
- Updated help for contributing to the project (#130, [@lipkau][])
- Documentation for the custom classes of the module (#107, [@lipkau][])
- Added full support for PowerShell Core (`pwsh`) (#119, [@lipkau][])
- Added AppVeyor tests on PowerShell v6 (Linux) (#119, [@lipkau][])
- Added AppVeyor tests on PowerShell v6 (Windows) (#119, [@lipkau][])
- Tests now run from `./Release` Path (#99, [@lipkau][])
- Have the Build script to "compile" the functions into the psm1 file (enhances performance) (#119, [@lipkau][])
- Have a zip file deploy as artifact of the release (#90, [@lipkau][])

## 2.2 - 2018-01-01

FEATURES:
- Automatic deployment of documentation to website (#120, [@lipkau][])
- New parameter `-Query` to `Get-Page` for complex searches (#106, [@lipkau][])
- Documentation for the custom classes of the module (#107, [@lipkau][])
- Added full support for PowerShell Core (`pwsh`) (#119, [@lipkau][])

IMPROVEMENTS:
- Fixed encoding of Unicode chars (#101, [@lipkau][])
- Require necessary Assembly for HttpUtility (#102, [@lipkau][])

## 2.1 - 2017-11-01

IMPROVEMENTS:
  - Shows a warning when the server requires a CAPTCHA for the authentication (#91, [@lipkau][])
  - Custom classes now print relevant data in `ToString()` (#92, [@lipkau][])

## 2.0 - 2017-08-17
A new major version! ConfluencePS has been totally refactored to introduce new features and greatly improve efficiency.

"A new major version" means limited older functionality was intentionally broken. In addition, there are a ton of good changes, so some big picture notes first:

- All functions changed from "Wiki" prefix to "Confluence", like `Get-ConfluencePage`
  - But the module accommodates for any prefix you want, e.g. `Import-Module ConfluencePS -Prefix Wiki`
- Functions changed or removed:
  - `Get-WikiLabelApplied` [removed; functionality added to `Get-ConfluencePage -Label foo`]
  - `Get-WikiPageLabel` > `Get-ConfluenceLabel`
  - `New-WikiLabel` > `Add-ConfluenceLabel`
- `Get-*` functions now support paging, and defining your preferred page size
- `-Limit` and `-Expand` parameters were removed from functions
  - With paging implementation, modifying the returned object limit isn't necessary
  - And allows for richer objects to be returned by default
- `-ApiUri` and `-Credential` parameters added to every function
  - This is useful if you have more than one Confluence instance
  - `Set-ConfluenceInfo` now defines `ApiUri` and `Credential` defaults for the current session
  - And you can override any single function:
  - `Get-ConfluenceSpace -ApiUri 'https://wiki2.example.com' -Credential (Get-Credential)`
- All functions now output custom object types, like `[ConfluencePS.Page]`
  - Allows for returning more object properties...
  - ...and only displaying the most relevant in the default output
  - Also enables a much improved pipeline flow
  - This behavior removed the need for the `-Expand` parameter
- Private functions are leveraged heavily to reduce repeat code
  - `Invoke-Method` is the most prominent example

If you like drinking from the firehose, here's [everything we closed for 2.0], because we probably forgot to list something here. Otherwise, read on for summarized details.

### Added
- All `Get-*` functions now support paging
- `-ApiUri` and `-Credential` parameters added to functions
  - `Set-ConfluenceInfo` behavior is mostly unchanged (see below)
- Objects returned are now custom typed, like `[ConfluencePS.Page]`
  - Try piping ConfluencePS objects into `Format-List *` to see all properties

### Changed
- Function prefix defaults to "Confluence" instead of "Wiki" (`Get-ConfluenceSpace`)
  - If you like "Wiki", you can `Import-Module ConfluencePS -Prefix Wiki`
- `Add-ConfluenceLabel`
  - Name used to be `New-WikiLabel`
  - The "Add" verb better reflects the function's behavior
- `Get-ConfluenceChildPage`
  - Default behavior returns only immediate child pages. Which also means...
  - Added `-Recurse` to return all pages below the given page, not just immediate child objects
    - NOTE: Recurse is not available in on-prem installs right now, only Atlassian cloud instances
  - `-ParentID` > `-PageID`
- `Get-ConfluenceLabel`
  - Name used to be `Get-WikiPageLabel`
  - Now returns `[ConfluencePS.ContentLabelSet]` objects
    - Which are relationships of `[ConfluencePS.Label]` & `[ConfluencePS.Page]` objects
- `Get-ConfluencePage`
  - `Get-ConfluencePage` (with no parameters) doesn't work anymore
    - With paging supported, this would be a ton of pages
    - `Get-ConfluenceSpace | Get-ConfluencePage` still works, if you really need it
  - Now returns `[ConfluencePS.Page]` objects
  - New `-Label` parameter filters returned pages by applied label(s)
  - New `-Space` parameter accepts Space objects
- `Get-ConfluenceSpace`
  - Now returns `[ConfluencePS.Space]` objects
  - `-Key` renamed to `-SpaceKey` ("Key" still works as an alias)
  - `-Name` parameter removed
- `New-ConfluencePage`
  - New `-Parent` parameter accepts Page objects
  - New `-Space` parameter accepts Space objects
- `New-ConfluenceSpace`
  - `-Key` renamed to `-SpaceKey` ("Key" still works as an alias)
- `Set-ConfluenceInfo`
  - Now adds the URI/Credential to `$PSDefaultParameterValues`
    - `-ApiUri` & `-Credential` parameters now exist on every function
    - `Set-ConfluenceInfo` defines their defaults for the current session
    - Meaning they could still be overwritten on any single command
  - No longer automatically prompts for credentials if `-Credential` is absent
    - Allows for anonymous authentication to public instances
  - New `-PromptCredentials` parameter displays a `Get-Credential` dialog while connecting
  - New `-PageSize` parameter optionally defines default page size for the session
- `Set-ConfluencePage`
  - Now returns `[ConfluencePS.Page]` objects
  - `-CurrentVersion` parameter removed (determined and incremented automatically now)
  - New `-Parent` parameter accepts Page objects

### Removed
- `-Limit` and `-Expand` parameters
  - `Get-*` function paging removes the need for fiddling with returned object limits
  - Custom object types hold relevant properties, removing the need to manually "expand" results
- `Get-WikiLabelApplied`
  - Functionality replaced with `Get-ConfluencePage -Label foo`

### Much ❤
[@lipkau](https://github.com/lipkau) refactored the entire module, and is the only reason `2.0` is a reality. In short, he is amazing. Thank you!


## 1.0.0-69 - 2016-11-28
No changelog available for version `1.0` of ConfluencePS. `1.0` was created in late 2015. Version `.69` was published to the PowerShell Gallery in Nov 2016, and it remained unchanged until `2.0`. If you're looking for things that changed prior to `2.0`...sorry, but these probably aren't the droids you're looking for. :)


[everything we closed for 2.0]: https://github.com/AtlassianPS/ConfluencePS/issues?utf8=%E2%9C%93&q=closed%3A2017-04-01..2017-08-17
[@alexsuslin]: https://github.com/alexsuslin
[@axxelG]: https://github.com/axxelG
[@beaudryj]: https://github.com/beaudryj
[@brianbunke]: https://github.com/brianbunke
[@Clijsters]: https://github.com/Clijsters
[@colhal]: https://github.com/colhal
[@Dejulia489]: https://github.com/Dejulia489
[@ebekker]: https://github.com/ebekker
[@jkknorr]: https://github.com/jkknorr
[@JohnAdders]: https://github.com/JohnAdders
[@kittholland]: https://github.com/kittholland
[@LiamLeane]: https://github.com/LiamLeane
[@lipkau]: https://github.com/lipkau
[@lukhase]: https://github.com/lukhase
[@padgers]: https://github.com/padgers
[@ThePSAdmin]: https://github.com/ThePSAdmin
