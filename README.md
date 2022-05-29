# Executable Notebook Template

This template contains a GitHub action to automate and scheduling execution of Jupyter notebooks. It only works with conda-environment. 

## Steps

* Upload a single notebook `*.ipynb` file. For instance, you can use the templates by theme (Exploration, Preprocessing, Modelling, Postprocessing)  provided by the Environmental Data Science book repository.
* Modify the config file with the notebook file name.
* Upload a conda environment.yml file containing the libraries to run the executable notebook.

After the above steps, the GitHub actions will succesfully generate notebook output cells in the `render` branch.

## Badges template
The badges below should be adapted according to the new notebook.

<div align="center">
    <h1>[Notebook title]</h1>
</div>

<p align="center">
    <a href="https://github.com/Environmental-DS-Book/template-executable-notebook/blob/main/LICENSE">
        <img alt="License" src="https://img.shields.io/badge/License-MIT-yellow.svg">
    </a>
    <a href="https://mybinder.org/v2/gh/NordicESMHub/pangeo_CMIP6_masked_and_weighted_average/main?labpath=masked_and_weighted_average.ipynb">
        <img alt="Binder" src="https://mybinder.org/badge_logo.svg">
    </a>
    <a href="https://github.com/Environmental-DS-Book/template-executable-notebook/actions/workflows/publish.yml/badge.svg">
        <img alt="Continuous integration badge" src="https://github.com/Environmental-DS-Book/template-executable-notebook/actions/workflows/publish.yml/badge.svg">
    </a>
    <br/>
</p>

<p align="center"> 
    <a href="https://w3id.org/ro-id/03dc6999-4799-4739-b184-0a9d11c50aa4">
        <img alt="RoHub" src="https://img.shields.io/badge/RoHub-FAIR_Executable_Research_Object-2ea44f?logo=Open+Access&logoColor=blue">
    </a>
</p>
