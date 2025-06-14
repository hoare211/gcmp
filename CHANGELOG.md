# Changelog
All notable changes to this project will be documented in this file.

[1.1.0] - 2024-05-21
### Changed
- **Major Workflow Improvement for `work` Keyword**: The script no longer creates a temporary copy when comparing the working directory (`work`). It now uses the live repository path directly. This provides two key benefits:
    - **In-place Editing**: Users can now modify files directly within the comparison tool, and the changes are saved to the actual working files.
    - **Increased Performance**: Eliminates the overhead of copying the entire working directory.
- The script's console output has been updated to clearly state when it's using a "live directory" versus a temporary "snapshot".

### Added
- A new validation check that prevents the script from comparing `work` with itself.

[1.0.0] - 2023-10-27
### Added
- Initial release of gcmp.
- Support for comparing work, stage, stash, and any commit-ish reference.
- Automatic cleanup of temporary directories.
- Customizable DIFF_TOOL variable.
- Comprehensive help message via --help.