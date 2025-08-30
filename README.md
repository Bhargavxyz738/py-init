# pinit-cli: A Structured Python Project and Dependency Manager

<p>
  <a href="https://pypi.org/project/pinit-cli/"><img alt="PyPI Version" src="https://img.shields.io/pypi/v/pinit-cli?color=blue"></a>
  <a href="https://github.com/Bhargavxyz738/py-init/blob/main/LICENSE"><img alt="License" src="https://img.shields.io/pypi/l/pinit-cli"></a>
  <a href="https://pypi.org/project/pinit-cli/"><img alt="Supported Python Versions" src="https://img.shields.io/pypi/pyversions/pinit-cli"></a>
</p>

`pinit-cli` is a command-line utility designed to enforce structure and automate the lifecycle of Python projects. It addresses the common procedural overhead involved in project initialization, dependency management, and script execution by providing a single, consistent interface. By abstracting away manual `venv` handling and `pip` command intricacies, `pinit` allows developers to focus on application logic rather than environment configuration.

The tool operates on a dual-file dependency system, utilizing a `project.json` manifest for declarative, high-level package management and a `requirements.txt` file as a concrete lockfile for ensuring reproducible environments.

## Core Philosophy and Design

The development of `pinit` is guided by several core principles aimed at improving the standard Python workflow:

1.  **Declarative Management**: Projects should declare their direct dependencies in a clear manifest (`project.json`). The full dependency graph, an implementation detail, should be managed automatically in a lockfile (`requirements.txt`) for reproducibility. This separates intent from outcome.

2.  **Environment Abstraction**: The developer should not need to manually activate or manage the virtual environment for routine tasks. The project's toolchain should be intrinsically aware of its environment, ensuring all commands execute in the correct context.

3.  **Deterministic Dependency Resolution**: Dependency removal should be as safe and intelligent as dependency addition. The tool must prevent the accidental removal of shared dependencies, thereby preserving the integrity of the project's environment.

4.  **Standardization**: Common project tasks such as running, testing, or linting should be standardized through a script runner, making projects more portable and easier for new contributors to engage with.

---

## In-Depth Feature Overview

### Project Initialization and Scaffolding
`pinit` automates the creation of a standardized project structure. The `pinit` command initiates an interactive sequence that configures the project's foundational elements, including:
*   **Directory Structure**: Establishes the project root and prepares it for version control and development.
*   **Virtual Environment**: Creates an isolated Python environment within a `venv` directory, a standard practice for dependency sandboxing.
*   **Manifest and Lockfile**: Generates `project.json` to record project metadata and direct dependencies, and `requirements.txt` to lock the versions of all installed packages.
*   **Entry Point**: Creates a default main script file to provide an immediate starting point for development.

### Automated Virtual Environment Integration
A central feature of `pinit` is its implicit handling of the project's virtual environment. For all core operations (`install`, `uninstall`, `run`), `pinit` automatically detects and utilizes the Python executable located within the local `venv` directory. This design choice provides significant benefits:
*   **Contextual Execution**: Ensures that all package management operations and script executions use the project-specific interpreter and its associated dependencies.
*   **Elimination of Manual Activation**: Frees the developer from needing to run `source venv/bin/activate` or `venv\Scripts\activate` before interacting with the project, reducing friction and potential errors.

### Advanced Dependency Management with Safety Guarantees
`pinit` introduces a more robust approach to dependency management than direct `pip` usage.

#### Intelligent Uninstall
The `pinit uninstall` command is designed to prevent common dependency conflicts that arise from manual package removal. Its process is as follows:
1.  **Dependency Graph Analysis**: It constructs the full dependency trees for both the package(s) being uninstalled and all other direct dependencies listed in `project.json`.
2.  **Intersection Calculation**: It calculates the set intersection of these dependency trees. Packages found in the intersection are identified as shared dependencies.
3.  **Targeted Removal**: It proceeds to uninstall only those packages that are exclusive to the dependency tree of the package being removed. Shared dependencies are preserved, and the user is informed of which packages were kept and why.

This mechanism ensures that removing a package does not inadvertently break other components of the application.

### Integrated Script Runner
The `project.json` manifest serves as a central registry for project-related commands, similar to the `scripts` field in `package.json`.
*   **Portability**: Scripts are defined declaratively, ensuring that any developer can execute standard project tasks with a consistent command (`pinit run <script-name>`).
*   **Environment-Awareness**: When a script begins with `python`, `pinit` automatically substitutes it with the full path to the `venv`'s Python executable, guaranteeing the script runs in the correct isolated environment.
*   **Parameterization**: Supports passing additional command-line arguments directly to the underlying script, allowing for flexible and dynamic command execution.

### Integrated Shell Management
For interactive tasks, such as using a debugger or experimenting with a REPL, `pinit shell` provides a direct entry point into an activated virtual environment. This command launches a new sub-shell with the environment fully sourced, providing a convenient and explicit way to work within the project's context when needed.

---

## Installation and System Requirements

`pinit-cli` is distributed via PyPI and can be installed globally using `pip`.

```bash
pip install pinit-cli
```

The source code is available on GitHub: [https://github.com/Bhargavxyz738/py-init.git](https://github.com/Bhargavxyz738/py-init.git)

#### Platform Compatibility
*   **Primary Support**: The tool is primarily developed and tested on **Windows 10 and newer**, where it is guaranteed to provide a stable and complete feature set.
*   **Functional Compatibility**: It is designed to be compatible with POSIX-compliant systems such as **macOS and Linux**. While all core functionality is expected to work, platform-specific edge cases may exist. Bug reports and contributions for improving cross-platform support are welcome.

---

## Comprehensive Usage Reference

### `pinit`
Initializes a new project in the current directory via an interactive setup wizard.

### `pinit install [<package-specifier>...]`
Manages the installation of project dependencies.
*   **To add new packages:**
    ```bash
    # Install the latest version of 'requests' and a specific version of 'typer'
    pinit install requests "typer==0.4.1"
    ```
*   **To restore an environment:** Executing the command without arguments will install all dependencies listed in `requirements.txt` or `project.json`. This is the standard procedure after cloning a project repository.
    ```bash
    pinit install
    ```

### `pinit uninstall <package-name> | --all`
Manages the removal of project dependencies.
*   **To remove specific packages safely:**
    ```bash
    pinit uninstall fastapi
    ```
*   **To remove all project dependencies:** This command uninstalls every package listed in `requirements.txt` and clears the dependencies from `project.json`.
    ```bash
    pinit uninstall --all
    ```

### `pinit run <script-name> [additional-args...]`
Executes a command defined in the `scripts` section of `project.json`.
*   **Example `project.json`:**
    ```json
    {
      "scripts": {
        "start": "python main.py",
        "test": "pytest"
      }
    }
    ```
*   **Execution:**
    ```bash
    # Runs the 'start' script
    pinit run start

    # Runs the 'test' script and passes the '-v' flag to pytest
    pinit run test -v
    ```

### `pinit shell`
Launches a new command-line shell with the project's virtual environment activated.

---

## The `project.json` Manifest Schema

The `project.json` file is the declarative core of your project.

*   `projectName` (string): The name of the project.
*   `version` (string): The project's semantic version number.
*   `mainFile` (string): The designated primary script file for the project.
*   `scripts` (object): A key-value map where keys are script names and values are the shell commands to execute.
*   `dependencies` (object): A key-value map of directly installed packages and their locked versions.

## Guidelines for Contribution

Contributions in the form of bug reports, feature requests, or pull requests are welcome. Please refer to the project's issue tracker on GitHub for ongoing development and to report new issues.

## License

This project is licensed under the terms of the MIT License.
