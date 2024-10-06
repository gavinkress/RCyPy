# RCyPy
 Automate Building Python Virtual Environments with integrated C++ and R Capabilities  with full Dependency Management
 
________________________________________________________
-------------------------------------------------------
# Create Venv with C++ and R Capabilities 
_______________________________________________________
-------------------------------------------------------

- Name: RCyPy
- Homepage: https://github.com/gavinkress/RCyPy
- Author: Gavin Kress
- email: gavinkress@gmail.com
- Date: 10/05/2024
- version: 1.0.0
- readme: RCyPy.md
- Programming Language(s): Python, C++, R, Rust, PowerShell, Bash
- License: MIT License
- Operating System: OS Independent

----------------------------------------------------------------
## Follow Instructions, do not blindly run code
----------------------------------------------------------------

This workflow will ensure your global Python Environment will
have C++ and R Capabilities complete with pip-tools package
and dependency management.

It will then create a virtual environment with the same capabilities
and features. It does all of this without relying on black box
package managers and version truncation sometimes associated
with common dependency management solutions like Poetry. It allows you to 
list other venvs with dependencies to import as well as additional packages to install.

Simply specify your parameters and Ensure you have the Language compilers 
at before you start and follow the quick workflow.

Parameters:
- VenvPath: Path to the RCyPyVenv to be created (string)
- Venvs: List of paths to all environments with dependencies to add to RCyPyVenv (array(string))
- addpkgs: Additional packages to install in RCyPyVenv (array(string))
- VenvPrompt: Prompt for Venv creation (string)
- Rprofiletextinput: .Rprofile configurations (string)
- RConfigpath: .Rprofile Path (string)


## Dependencies: *Manually Install the following*
--------------------------------------------------------------------------------

- [Install Rust Compiler](https://www.rust-lang.org/tools/install)

- [Install R](https://cran.r-project.org/bin/windows/base/)
 
- [Install Rtools](https://cran.r-project.org/bin/windows/Rtools/rtools44/rtools.html) 

- [Install C++/.NET Build Tools](https://visualstudio.microsoft.com/visual-cpp-build-tools/)

- - Its easiest to keep all versions supported through the VS Installer, since many libraries and applications require legacy builds, you could get by with installing only when you get the error(s):

- - - distutils.errors.DistutilsExecError: command failed with exit code 1181
- - - LINK : fatal error LNK1181: cannot open input file (for certain versions on non-UNIX OS we need to build some .lib files manually)

- [Optimize and Repair User and System Paths and Variables](https://github.com/gavinkress/Repair-Broken-Envs_Path)

## User and System Path and Variable Modifications: *Manually Specify then restart shell* 
-----------------------------------------------------------------------------------------
```powershell

### Target Venv Creation Path
cd ~/
$VenvPath = "$env:USERPROFILE\OneDrive\Centralized Programming Heirarchy\.env\.virtualenvs\RCyPyVenv"

### List paths to all environments with needed dependencies to add to RCyPyVenv
$Venvs = @(
    "$env:USERPROFILE\OneDrive\Centralized Programming Heirarchy\.env\.virtualenvs\GeneralPythonVenv\" 
    "$env:USERPROFILE\OneDrive\Centralized Programming Heirarchy\_Env configurations and Package managers\GeneralVenv\"
)

### Additional packages to install in RCyPyVenv
$addpkgs = @(
	"torch",
	"torchaudio",
	"torchvision"
)

### Promt for Venv: "." dynamically sets cwd as base path when Venv is activated
$VenvPrompt = "."

### .Rprofile Configurations
$Rprofiletextinput = @"
options("digits.secs"=3)            # show sub-second time stamps
r <- getOption("repos")             # hard code the US repo for CRAN
r["CRAN"] <- "http://cran.us.r-project.org"
options(repos = r)
rm(r)
"@

### .Rprofile Path
$RConfigpath = "$env:USERPROFILE\OneDrive\Centralized Programming Heirarchy\.env\R\.Rprofile"

### Function to write output in a pretty format
function WriteOutputPretty {
	param (
		[string]$currentmsg
	)
	$n = $currentmsg.length
	$outers = "------------------------------------------------------------------------------------------"+"-"*$n
	echo ""
	echo $outers
	echo "-------------------------------------------- $currentmsg --------------------------------------------"
	echo $outers
	echo ""
	
}
WriteOutputPretty -currentmsg "Attention!"
WriteOutputPretty -currentmsg "You should Repair, Optimize, and Build User and System Paths and Variables"
WriteOutputPretty -currentmsg "See GitHub.com/gavnnkress/Repair-Broken_Envs_Path"


Remove-Item $RConfigpath
echo $Rprofiletextinput  >> $RConfigpath


```

## R packages
---------------------
```powershell

cd ~/
R.exe

```

```R
install.packages('Rcpp', dependencies = TRUE)
install.packages("Rcmdr", dependencies = TRUE)
q()
```

## Rpy2 build packages (R.lib and m.lib)
----------------------------------------

### R.lib
```powershell

WriteOutputPretty -currentmsg "Building R.lib"
cd "C:\Program Files\R\R-4.4.1\bin\x64"
$dllPath = "./R.dll"  # Path to R.dll, adjust if necessary
$defPath = "./R.def"  # Output R.def file


#### Functions to include
$defContent = @"
LIBRARY R
EXPORTS`n
"@

#### Add R functions to content
$dumpbinOutput = & dumpbin /EXPORTS $dllPath
$dumpbinOutput -split "`n" | ForEach-Object {
    $line = $_.Trim()
    ##### Split by space, filter lines that have at least 4 elements and start with ordinal number
    $parts = $line -split '\s+' 
    if (($parts.Length -ge 4 -and $parts[0] -match '^\d+$') -and $parts[-1] -notin "names", "functions") {
        $functionName = $parts[-1]
        $defContent += "$functionName`n"
    }
}

#### Write to file and create library
Remove-Item $defPath
$defContent | Set-Content -Path $defPath
lib /DEF:R.def /OUT:R.lib /MACHINE:X64

echo $defContent
WriteOutputPretty -currentmsg "R.lib Built Successfully"

```

### m.lib
```powershell

WriteOutputPretty -currentmsg "Building R.lib"
cd "C:\Program Files\R\R-4.4.1\bin\x64"
$defPath = "./m.def" 

$defContent = @"
LIBRARY m
EXPORTS`n
"@

#### Define paths to the DLLs to look for math functions
$DllPaths = @()
$basepath = "C:\Windows\System32\"
foreach ($substr in @("ucrt", "msvcrt")){
	Get-ChildItem -Path $basepath -Filter "*.dll" | ForEach-Object {
		if ($_.Name -match "^.*$substr.*$") {
			$DllPaths += $_.FullName
		}
	}
}


#### Define the list of math function substrings to look for
$mathFunctions = @(
    "sin",
    "cos",
    "tan",
    "sqrt",
    "exp",
    "log",
    "log10",
    "pow",
    "ceil",
    "floor",
    "fabs",
    "fmod",
    "atan",
    "atan2",
    "asin",
    "acos",
    "sinh",
    "cosh",
    "tanh",
    "asinh",
    "acosh",
    "atanh",
    "hypot",
    "round",
    "trunc",
    "modf",
    "ldexp",
    "frexp",
    "expm1",
    "log1p",
    "erf",
    "erfc",
    "gamma",
    "lgamma"
)


#### Add math functions to content
foreach ($dllpath in $DllPaths) {
		$dumpbinOutput = & dumpbin /EXPORTS $dllPath
		$dumpbinOutput -split "`n" | ForEach-Object {
		$line = $_.Trim()
		##### Split by space, filter lines that have at least 4 elements and start with ordinal number and match math functions
		$parts = $line -split '\s+' 
		$fname = $parts[-1]
		if ((($parts.Length -ge 4 -and $parts[0] -match '^\d+$') -and $parts[-1] -notin "names", "functions") -and [bool]($mathFunctions | Where-Object { $fname -match "^.*$_.*$" }))  {
			$defContent += "$fname`n"
		}
	}
}

#### Write to file and create library
Remove-Item $defPath
$defContent | Set-Content -Path $defPath
lib /DEF:m.def /OUT:m.lib /MACHINE:X64

echo $defContent
WriteOutputPretty -currentmsg "m.lib Built Successfully"

```

## Global Python Dependency Packages (Core, C++, R)
-----------------------------------------------------
### Update Packages and Create Distribution Wheels
```powershell

WriteOutputPretty -currentmsg "Building Global Python Dependency Packages (Core, C++, R)"

function UpdateCreateDist {
	echo "Updating and Creating Distribution Wheels...."
	pip freeze | %{$_.split('==')[0]} | %{pip install --upgrade $_}
	Remove-Item ./dist/ -r 
	pip cache purge
	mkdir ./dist/
	pip freeze > ./dist/requirements.in
	pip-compile ./dist/requirements.in --upgrade --output-file ./dist/requirements.txt
	pip-sync ./dist/requirements.txt

	mkdir ./dist/dependencies/
	pip download -r requirements.txt -d ./dist/dependencies/
	tar -czvf ./dist/dependencies_whls.tar.gz --exclude=./dist/dependencies_whls.tar.gz ./dist/dependencies/*.whl
 
	$recstext = Get-Content ./dist/requirements.txt 
	$recstext = @($recstext -split '`n' | Where-Object{ $_ -match "^.*==.*$"})
	$recstext = $recstext | %{$broken = $_ -split '=='; '"{0}"' -f $broken[0]} 
	$recstext  = $recstext -join ", " 
 
	$metadatain = @"
	[build-system]
	requires = [$recstext]
 	[project]
	name = 'RCyPy'
	version = '1.0.0'
	authors = [{ name='Gavin Kress', email='gavinkress@gmail.com'}]
	description = 'RCyPy Environment'
	readme = 'RCyPy.md'
	requires-python = '>=3.12'
	classifiers = [
	'Programming Language :: Python :: 3',
	'License :: OSI Approved :: MIT License',
	'Operating System :: OS Independent'
	]
# 
	[project.urls]
	Homepage = 'https://github.com/gavinkress/RCyPy'
	"@


	Remove-Item ./dist/pyproject.toml
	$metadatain | Out-File -FilePath ./dist/pyproject.toml -Encoding utf8
	cd ./dist/
	python -m build
	echo "Update Complete and Distribution Wheels Created"
	cd ./..
}

### Core Packages
pip install pipx
pipx install Poetry
pipx ensurepath
pipx upgrade poetry

function InstallCorePackages {
	echo "Installing Core Packages...."
	$corepkgs = @(
		"pip",
		"setuptools",
		"wheel",
		"flask",
		"flake8",
		"build",
		"pip-tools",
		"virtualenv"
	)
	foreach ($corepkg in $corepkgs){
		pip install --upgrade $corepkg --verbose
		python -m pip install --upgrade virtualenv --verbose
	}
}

### C++ and R Interpreter Packages
function InstallInterpreterPackages {
	echo "Installing C++ and R Interpreter Packages...."
	$interpreterpkgs = @(
		"cffi",
		"cython",
		"libcpp",
		"rpy2",
		"rpy2[all]",
		"rpy2[test]",
		"rpy2.rinterface_lib",
		"rpy2.rinterface"
	)
	foreach ($intpkg in $interpreterpkgs){
		pip install --upgrade $intpkg --verbose
		python -m pip install --upgrade virtualenv --verbose
	}
}

cd ~/
InstallCorePackages
UpdateCreateDist
InstallInterpreterPackages
UpdateCreateDist


WriteOutputPretty -currentmsg "Global Python Dependency Packages (Core, C++, R) Built Successfully"

```

## Tests
------------
```powershell

WriteOutputPretty -currentmsg "Testing"

pytest ./tests --cov rpy2.rinterface_lib --cov rpy2.rinterface --cov rpy2.ipython pytest --cov rpy2.robject 

WriteOutputPretty -currentmsg "Tests Passed Successfully"
```

## Ensure existance of baseline packages and Requirements.txt in other Venvs
-----------------------------------------------------------------------------
```powershell

WriteOutputPretty -currentmsg "Building core packages and requirements in venvs to import"
for ( $index = 0; $index -lt $Venvs.count; $index++){
	cd $($Venvs[$index])
	./Scripts/activate
	InstallCorePackages
	UpdateCreateDist
	deactivate
}

WriteOutputPretty -currentmsg "Core packages and requirements in venvs built successfully"
```

## Create RCyPyVenv Base 
-----------------------
```powershell

WriteOutputPretty -currentmsg "Creating RCyPyVenv base at $VenvPath"
cd ~/
python -m venv $VenvPath --system-site-packages --copies  --prompt $($VenvPrompt)
python -m venv $VenvPath --system-site-packages --copies --upgrade --prompt $($VenvPrompt) --upgrade-deps 

cd $($VenvPath)
./Scripts/activate
InstallCorePackages
UpdateCreateDist

WriteOutputPretty -currentmsg "RCyPyVenv base created successfully"
```

## Build RCyPyVenv Dependencies 
------------------------------
```powershell

WriteOutputPretty -currentmsg "Building RCyPyVenv dependencies"

### Get dependencies
Remove-Item ./temp/ -r 
mkdir ./temp/
Copy-Item -Path ./dist/requirements.txt -Destination ./temp/requirements_current.in
Copy-Item -Path ~/dist/requirements.txt -Destination ./temp/requirements_base.in
for ( $index = 0; $index -lt $Venvs.count; $index++){
	Copy-Item -Path "$($Venvs[$index])\requirements.txt" -Destination ./temp/requirements_$($index+1).in
}
$requirements_arr = ls ./temp/ | %{$_.Name} | sort -Descending
$requirements_str = $requirements_arr -join " ./temp/"

### Install dependencies
Remove-Item ./dist/ -r 
mkdir ./dist/
pip-compile $requirements_str --output-file ./dist/requirements.txt
pip-sync ./dist/requirements.txt
UpdateCreateDist
Remove-Item ./temp/ -r

installInterpreterPackages
UpdateCreateDist

### Install more packages  
foreach ($addpkg in $addpkgs){
	pip install --upgrade $addpkg --verbose
}
UpdateCreateDist

WriteOutputPretty -currentmsg "RCyPy dependencies built successfully"

```

-----------------------------------------------------------------------------------------------------------------------------
## Make sure all your workspace and user settings are updated in vs/vscode
## and save these instructions in the venv as .md for best view
-----------------------------------------------------------------------------------------------------------------------------
