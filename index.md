# Using masks and computing weighted average

This example is based from xarray example http://xarray.pydata.org/en/stable/examples/area_weighted_temperature.html

## Import python packages


```python
pip install nc-time-axis
```

    Requirement already satisfied: nc-time-axis in /usr/share/miniconda/envs/repo/lib/python3.10/site-packages (1.4.1)
    Requirement already satisfied: matplotlib in /usr/share/miniconda/envs/repo/lib/python3.10/site-packages (from nc-time-axis) (3.5.2)
    Requirement already satisfied: cftime>=1.5 in /usr/share/miniconda/envs/repo/lib/python3.10/site-packages (from nc-time-axis) (1.6.0)
    Requirement already satisfied: numpy in /usr/share/miniconda/envs/repo/lib/python3.10/site-packages (from nc-time-axis) (1.22.4)
    Requirement already satisfied: pyparsing>=2.2.1 in /usr/share/miniconda/envs/repo/lib/python3.10/site-packages (from matplotlib->nc-time-axis) (3.0.9)
    Requirement already satisfied: python-dateutil>=2.7 in /usr/share/miniconda/envs/repo/lib/python3.10/site-packages (from matplotlib->nc-time-axis) (2.8.2)
    Requirement already satisfied: pillow>=6.2.0 in /usr/share/miniconda/envs/repo/lib/python3.10/site-packages (from matplotlib->nc-time-axis) (9.1.1)
    Requirement already satisfied: fonttools>=4.22.0 in /usr/share/miniconda/envs/repo/lib/python3.10/site-packages (from matplotlib->nc-time-axis) (4.33.3)
    Requirement already satisfied: packaging>=20.0 in /usr/share/miniconda/envs/repo/lib/python3.10/site-packages (from matplotlib->nc-time-axis) (21.3)
    Requirement already satisfied: kiwisolver>=1.0.1 in /usr/share/miniconda/envs/repo/lib/python3.10/site-packages (from matplotlib->nc-time-axis) (1.4.2)
    Requirement already satisfied: cycler>=0.10 in /usr/share/miniconda/envs/repo/lib/python3.10/site-packages (from matplotlib->nc-time-axis) (0.11.0)
    Requirement already satisfied: six>=1.5 in /usr/share/miniconda/envs/repo/lib/python3.10/site-packages (from python-dateutil>=2.7->matplotlib->nc-time-axis) (1.16.0)
    Note: you may need to restart the kernel to use updated packages.



```python
import xarray as xr
xr.set_options(display_style='html')
import intake
import cftime
import matplotlib.pyplot as plt
import cartopy.crs as ccrs
import numpy as np
%matplotlib inline
```


```python
cat_url = "https://storage.googleapis.com/cmip6/pangeo-cmip6.json"
col = intake.open_esm_datastore(cat_url)
col
```


<p><strong>pangeo-cmip6 catalog with 7686 dataset(s) from 514961 asset(s)</strong>:</p> <div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>unique</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>activity_id</th>
      <td>18</td>
    </tr>
    <tr>
      <th>institution_id</th>
      <td>36</td>
    </tr>
    <tr>
      <th>source_id</th>
      <td>88</td>
    </tr>
    <tr>
      <th>experiment_id</th>
      <td>170</td>
    </tr>
    <tr>
      <th>member_id</th>
      <td>657</td>
    </tr>
    <tr>
      <th>table_id</th>
      <td>37</td>
    </tr>
    <tr>
      <th>variable_id</th>
      <td>700</td>
    </tr>
    <tr>
      <th>grid_label</th>
      <td>10</td>
    </tr>
    <tr>
      <th>zstore</th>
      <td>514961</td>
    </tr>
    <tr>
      <th>dcpp_init_year</th>
      <td>60</td>
    </tr>
    <tr>
      <th>version</th>
      <td>737</td>
    </tr>
  </tbody>
</table>
</div>


## Search data


```python
cat = col.search(source_id=['NorESM2-LM'], experiment_id=['historical'], table_id=['Amon'], variable_id=['tas'], member_id=['r1i1p1f1'])
cat.df
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>activity_id</th>
      <th>institution_id</th>
      <th>source_id</th>
      <th>experiment_id</th>
      <th>member_id</th>
      <th>table_id</th>
      <th>variable_id</th>
      <th>grid_label</th>
      <th>zstore</th>
      <th>dcpp_init_year</th>
      <th>version</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>CMIP</td>
      <td>NCC</td>
      <td>NorESM2-LM</td>
      <td>historical</td>
      <td>r1i1p1f1</td>
      <td>Amon</td>
      <td>tas</td>
      <td>gn</td>
      <td>gs://cmip6/CMIP6/CMIP/NCC/NorESM2-LM/historica...</td>
      <td>NaN</td>
      <td>20190815</td>
    </tr>
  </tbody>
</table>
</div>



## Create dictionary from the list of datasets we found
- This step may take several minutes so be patient!


```python
dset_dict = cat.to_dataset_dict(zarr_kwargs={'use_cftime':True})
```

    
    --> The keys in the returned dictionary of datasets are constructed as follows:
    	'activity_id.institution_id.source_id.experiment_id.table_id.grid_label'




<style>
    /* Turns off some styling */
    progress {
        /* gets rid of default border in Firefox and Opera. */
        border: none;
        /* Needs to be in here for Safari polyfill so background images work as expected. */
        background-size: auto;
    }
    .progress-bar-interrupted, .progress-bar-interrupted::-webkit-progress-bar {
        background: #F44336;
    }
</style>





<div>
  <progress value='1' class='' max='1' style='width:300px; height:20px; vertical-align: middle;'></progress>
  100.00% [1/1 00:00<00:00]
</div>




```python
list(dset_dict.keys())
```




    ['CMIP.NCC.NorESM2-LM.historical.Amon.gn']




```python
dset = dset_dict[list(dset_dict.keys())[0]]
dset
```




<div><svg style="position: absolute; width: 0; height: 0; overflow: hidden">
<defs>
<symbol id="icon-database" viewBox="0 0 32 32">
<path d="M16 0c-8.837 0-16 2.239-16 5v4c0 2.761 7.163 5 16 5s16-2.239 16-5v-4c0-2.761-7.163-5-16-5z"></path>
<path d="M16 17c-8.837 0-16-2.239-16-5v6c0 2.761 7.163 5 16 5s16-2.239 16-5v-6c0 2.761-7.163 5-16 5z"></path>
<path d="M16 26c-8.837 0-16-2.239-16-5v6c0 2.761 7.163 5 16 5s16-2.239 16-5v-6c0 2.761-7.163 5-16 5z"></path>
</symbol>
<symbol id="icon-file-text2" viewBox="0 0 32 32">
<path d="M28.681 7.159c-0.694-0.947-1.662-2.053-2.724-3.116s-2.169-2.030-3.116-2.724c-1.612-1.182-2.393-1.319-2.841-1.319h-15.5c-1.378 0-2.5 1.121-2.5 2.5v27c0 1.378 1.122 2.5 2.5 2.5h23c1.378 0 2.5-1.122 2.5-2.5v-19.5c0-0.448-0.137-1.23-1.319-2.841zM24.543 5.457c0.959 0.959 1.712 1.825 2.268 2.543h-4.811v-4.811c0.718 0.556 1.584 1.309 2.543 2.268zM28 29.5c0 0.271-0.229 0.5-0.5 0.5h-23c-0.271 0-0.5-0.229-0.5-0.5v-27c0-0.271 0.229-0.5 0.5-0.5 0 0 15.499-0 15.5 0v7c0 0.552 0.448 1 1 1h7v19.5z"></path>
<path d="M23 26h-14c-0.552 0-1-0.448-1-1s0.448-1 1-1h14c0.552 0 1 0.448 1 1s-0.448 1-1 1z"></path>
<path d="M23 22h-14c-0.552 0-1-0.448-1-1s0.448-1 1-1h14c0.552 0 1 0.448 1 1s-0.448 1-1 1z"></path>
<path d="M23 18h-14c-0.552 0-1-0.448-1-1s0.448-1 1-1h14c0.552 0 1 0.448 1 1s-0.448 1-1 1z"></path>
</symbol>
</defs>
</svg>
<style>/* CSS stylesheet for displaying xarray objects in jupyterlab.
 *
 */

:root {
  --xr-font-color0: var(--jp-content-font-color0, rgba(0, 0, 0, 1));
  --xr-font-color2: var(--jp-content-font-color2, rgba(0, 0, 0, 0.54));
  --xr-font-color3: var(--jp-content-font-color3, rgba(0, 0, 0, 0.38));
  --xr-border-color: var(--jp-border-color2, #e0e0e0);
  --xr-disabled-color: var(--jp-layout-color3, #bdbdbd);
  --xr-background-color: var(--jp-layout-color0, white);
  --xr-background-color-row-even: var(--jp-layout-color1, white);
  --xr-background-color-row-odd: var(--jp-layout-color2, #eeeeee);
}

html[theme=dark],
body.vscode-dark {
  --xr-font-color0: rgba(255, 255, 255, 1);
  --xr-font-color2: rgba(255, 255, 255, 0.54);
  --xr-font-color3: rgba(255, 255, 255, 0.38);
  --xr-border-color: #1F1F1F;
  --xr-disabled-color: #515151;
  --xr-background-color: #111111;
  --xr-background-color-row-even: #111111;
  --xr-background-color-row-odd: #313131;
}

.xr-wrap {
  display: block !important;
  min-width: 300px;
  max-width: 700px;
}

.xr-text-repr-fallback {
  /* fallback to plain text repr when CSS is not injected (untrusted notebook) */
  display: none;
}

.xr-header {
  padding-top: 6px;
  padding-bottom: 6px;
  margin-bottom: 4px;
  border-bottom: solid 1px var(--xr-border-color);
}

.xr-header > div,
.xr-header > ul {
  display: inline;
  margin-top: 0;
  margin-bottom: 0;
}

.xr-obj-type,
.xr-array-name {
  margin-left: 2px;
  margin-right: 10px;
}

.xr-obj-type {
  color: var(--xr-font-color2);
}

.xr-sections {
  padding-left: 0 !important;
  display: grid;
  grid-template-columns: 150px auto auto 1fr 20px 20px;
}

.xr-section-item {
  display: contents;
}

.xr-section-item input {
  display: none;
}

.xr-section-item input + label {
  color: var(--xr-disabled-color);
}

.xr-section-item input:enabled + label {
  cursor: pointer;
  color: var(--xr-font-color2);
}

.xr-section-item input:enabled + label:hover {
  color: var(--xr-font-color0);
}

.xr-section-summary {
  grid-column: 1;
  color: var(--xr-font-color2);
  font-weight: 500;
}

.xr-section-summary > span {
  display: inline-block;
  padding-left: 0.5em;
}

.xr-section-summary-in:disabled + label {
  color: var(--xr-font-color2);
}

.xr-section-summary-in + label:before {
  display: inline-block;
  content: '►';
  font-size: 11px;
  width: 15px;
  text-align: center;
}

.xr-section-summary-in:disabled + label:before {
  color: var(--xr-disabled-color);
}

.xr-section-summary-in:checked + label:before {
  content: '▼';
}

.xr-section-summary-in:checked + label > span {
  display: none;
}

.xr-section-summary,
.xr-section-inline-details {
  padding-top: 4px;
  padding-bottom: 4px;
}

.xr-section-inline-details {
  grid-column: 2 / -1;
}

.xr-section-details {
  display: none;
  grid-column: 1 / -1;
  margin-bottom: 5px;
}

.xr-section-summary-in:checked ~ .xr-section-details {
  display: contents;
}

.xr-array-wrap {
  grid-column: 1 / -1;
  display: grid;
  grid-template-columns: 20px auto;
}

.xr-array-wrap > label {
  grid-column: 1;
  vertical-align: top;
}

.xr-preview {
  color: var(--xr-font-color3);
}

.xr-array-preview,
.xr-array-data {
  padding: 0 5px !important;
  grid-column: 2;
}

.xr-array-data,
.xr-array-in:checked ~ .xr-array-preview {
  display: none;
}

.xr-array-in:checked ~ .xr-array-data,
.xr-array-preview {
  display: inline-block;
}

.xr-dim-list {
  display: inline-block !important;
  list-style: none;
  padding: 0 !important;
  margin: 0;
}

.xr-dim-list li {
  display: inline-block;
  padding: 0;
  margin: 0;
}

.xr-dim-list:before {
  content: '(';
}

.xr-dim-list:after {
  content: ')';
}

.xr-dim-list li:not(:last-child):after {
  content: ',';
  padding-right: 5px;
}

.xr-has-index {
  font-weight: bold;
}

.xr-var-list,
.xr-var-item {
  display: contents;
}

.xr-var-item > div,
.xr-var-item label,
.xr-var-item > .xr-var-name span {
  background-color: var(--xr-background-color-row-even);
  margin-bottom: 0;
}

.xr-var-item > .xr-var-name:hover span {
  padding-right: 5px;
}

.xr-var-list > li:nth-child(odd) > div,
.xr-var-list > li:nth-child(odd) > label,
.xr-var-list > li:nth-child(odd) > .xr-var-name span {
  background-color: var(--xr-background-color-row-odd);
}

.xr-var-name {
  grid-column: 1;
}

.xr-var-dims {
  grid-column: 2;
}

.xr-var-dtype {
  grid-column: 3;
  text-align: right;
  color: var(--xr-font-color2);
}

.xr-var-preview {
  grid-column: 4;
}

.xr-var-name,
.xr-var-dims,
.xr-var-dtype,
.xr-preview,
.xr-attrs dt {
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  padding-right: 10px;
}

.xr-var-name:hover,
.xr-var-dims:hover,
.xr-var-dtype:hover,
.xr-attrs dt:hover {
  overflow: visible;
  width: auto;
  z-index: 1;
}

.xr-var-attrs,
.xr-var-data {
  display: none;
  background-color: var(--xr-background-color) !important;
  padding-bottom: 5px !important;
}

.xr-var-attrs-in:checked ~ .xr-var-attrs,
.xr-var-data-in:checked ~ .xr-var-data {
  display: block;
}

.xr-var-data > table {
  float: right;
}

.xr-var-name span,
.xr-var-data,
.xr-attrs {
  padding-left: 25px !important;
}

.xr-attrs,
.xr-var-attrs,
.xr-var-data {
  grid-column: 1 / -1;
}

dl.xr-attrs {
  padding: 0;
  margin: 0;
  display: grid;
  grid-template-columns: 125px auto;
}

.xr-attrs dt,
.xr-attrs dd {
  padding: 0;
  margin: 0;
  float: left;
  padding-right: 10px;
  width: auto;
}

.xr-attrs dt {
  font-weight: normal;
  grid-column: 1;
}

.xr-attrs dt:hover span {
  display: inline-block;
  background: var(--xr-background-color);
  padding-right: 10px;
}

.xr-attrs dd {
  grid-column: 2;
  white-space: pre-wrap;
  word-break: break-all;
}

.xr-icon-database,
.xr-icon-file-text2 {
  display: inline-block;
  vertical-align: middle;
  width: 1em;
  height: 1.5em !important;
  stroke-width: 0;
  stroke: currentColor;
  fill: currentColor;
}
</style><pre class='xr-text-repr-fallback'>&lt;xarray.Dataset&gt;
Dimensions:    (lat: 96, bnds: 2, lon: 144, member_id: 1, time: 1980)
Coordinates:
    height     float64 ...
  * lat        (lat) float64 -90.0 -88.11 -86.21 -84.32 ... 86.21 88.11 90.0
    lat_bnds   (lat, bnds) float64 dask.array&lt;chunksize=(96, 2), meta=np.ndarray&gt;
  * lon        (lon) float64 0.0 2.5 5.0 7.5 10.0 ... 350.0 352.5 355.0 357.5
    lon_bnds   (lon, bnds) float64 dask.array&lt;chunksize=(144, 2), meta=np.ndarray&gt;
  * time       (time) object 1850-01-16 12:00:00 ... 2014-12-16 12:00:00
    time_bnds  (time, bnds) object dask.array&lt;chunksize=(1980, 2), meta=np.ndarray&gt;
  * member_id  (member_id) &lt;U8 &#x27;r1i1p1f1&#x27;
Dimensions without coordinates: bnds
Data variables:
    tas        (member_id, time, lat, lon) float32 dask.array&lt;chunksize=(1, 990, 96, 144), meta=np.ndarray&gt;
Attributes: (12/54)
    Conventions:               CF-1.7 CMIP-6.2
    activity_id:               CMIP
    branch_method:             Hybrid-restart from year 1600-01-01 of piControl
    branch_time:               0.0
    branch_time_in_child:      0.0
    branch_time_in_parent:     430335.0
    ...                        ...
    variable_id:               tas
    variant_label:             r1i1p1f1
    netcdf_tracking_ids:       hdl:21.14100/2486cf87-033c-4848-ab3e-e828c3b7c...
    version_id:                v20190815
    intake_esm_varname:        [&#x27;tas&#x27;]
    intake_esm_dataset_key:    CMIP.NCC.NorESM2-LM.historical.Amon.gn</pre><div class='xr-wrap' style='display:none'><div class='xr-header'><div class='xr-obj-type'>xarray.Dataset</div></div><ul class='xr-sections'><li class='xr-section-item'><input id='section-685a6a29-bd4a-4353-ba2e-07fc79c9d38d' class='xr-section-summary-in' type='checkbox' disabled ><label for='section-685a6a29-bd4a-4353-ba2e-07fc79c9d38d' class='xr-section-summary'  title='Expand/collapse section'>Dimensions:</label><div class='xr-section-inline-details'><ul class='xr-dim-list'><li><span class='xr-has-index'>lat</span>: 96</li><li><span>bnds</span>: 2</li><li><span class='xr-has-index'>lon</span>: 144</li><li><span class='xr-has-index'>member_id</span>: 1</li><li><span class='xr-has-index'>time</span>: 1980</li></ul></div><div class='xr-section-details'></div></li><li class='xr-section-item'><input id='section-52798567-ef6d-4a64-80c1-b5b7e4a707fa' class='xr-section-summary-in' type='checkbox'  checked><label for='section-52798567-ef6d-4a64-80c1-b5b7e4a707fa' class='xr-section-summary' >Coordinates: <span>(8)</span></label><div class='xr-section-inline-details'></div><div class='xr-section-details'><ul class='xr-var-list'><li class='xr-var-item'><div class='xr-var-name'><span>height</span></div><div class='xr-var-dims'>()</div><div class='xr-var-dtype'>float64</div><div class='xr-var-preview xr-preview'>...</div><input id='attrs-81e28202-1330-4290-b26f-ce9657424a2d' class='xr-var-attrs-in' type='checkbox' ><label for='attrs-81e28202-1330-4290-b26f-ce9657424a2d' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-7434f8a3-21e4-4582-b041-eab6efe3e609' class='xr-var-data-in' type='checkbox'><label for='data-7434f8a3-21e4-4582-b041-eab6efe3e609' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'><dt><span>axis :</span></dt><dd>Z</dd><dt><span>long_name :</span></dt><dd>height</dd><dt><span>positive :</span></dt><dd>up</dd><dt><span>standard_name :</span></dt><dd>height</dd><dt><span>units :</span></dt><dd>m</dd></dl></div><div class='xr-var-data'><pre>array(2.)</pre></div></li><li class='xr-var-item'><div class='xr-var-name'><span class='xr-has-index'>lat</span></div><div class='xr-var-dims'>(lat)</div><div class='xr-var-dtype'>float64</div><div class='xr-var-preview xr-preview'>-90.0 -88.11 -86.21 ... 88.11 90.0</div><input id='attrs-bc7e92b2-2b1b-483a-8617-7512464064a4' class='xr-var-attrs-in' type='checkbox' ><label for='attrs-bc7e92b2-2b1b-483a-8617-7512464064a4' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-7c504875-0d5c-4706-a932-5d09b2141dff' class='xr-var-data-in' type='checkbox'><label for='data-7c504875-0d5c-4706-a932-5d09b2141dff' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'><dt><span>axis :</span></dt><dd>Y</dd><dt><span>bounds :</span></dt><dd>lat_bnds</dd><dt><span>long_name :</span></dt><dd>Latitude</dd><dt><span>standard_name :</span></dt><dd>latitude</dd><dt><span>units :</span></dt><dd>degrees_north</dd></dl></div><div class='xr-var-data'><pre>array([-90.      , -88.105263, -86.210526, -84.315789, -82.421053, -80.526316,
       -78.631579, -76.736842, -74.842105, -72.947368, -71.052632, -69.157895,
       -67.263158, -65.368421, -63.473684, -61.578947, -59.684211, -57.789474,
       -55.894737, -54.      , -52.105263, -50.210526, -48.315789, -46.421053,
       -44.526316, -42.631579, -40.736842, -38.842105, -36.947368, -35.052632,
       -33.157895, -31.263158, -29.368421, -27.473684, -25.578947, -23.684211,
       -21.789474, -19.894737, -18.      , -16.105263, -14.210526, -12.315789,
       -10.421053,  -8.526316,  -6.631579,  -4.736842,  -2.842105,  -0.947368,
         0.947368,   2.842105,   4.736842,   6.631579,   8.526316,  10.421053,
        12.315789,  14.210526,  16.105263,  18.      ,  19.894737,  21.789474,
        23.684211,  25.578947,  27.473684,  29.368421,  31.263158,  33.157895,
        35.052632,  36.947368,  38.842105,  40.736842,  42.631579,  44.526316,
        46.421053,  48.315789,  50.210526,  52.105263,  54.      ,  55.894737,
        57.789474,  59.684211,  61.578947,  63.473684,  65.368421,  67.263158,
        69.157895,  71.052632,  72.947368,  74.842105,  76.736842,  78.631579,
        80.526316,  82.421053,  84.315789,  86.210526,  88.105263,  90.      ])</pre></div></li><li class='xr-var-item'><div class='xr-var-name'><span>lat_bnds</span></div><div class='xr-var-dims'>(lat, bnds)</div><div class='xr-var-dtype'>float64</div><div class='xr-var-preview xr-preview'>dask.array&lt;chunksize=(96, 2), meta=np.ndarray&gt;</div><input id='attrs-a9b466cb-a610-485b-b25d-51f87bad91df' class='xr-var-attrs-in' type='checkbox' disabled><label for='attrs-a9b466cb-a610-485b-b25d-51f87bad91df' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-7d7bab23-bc74-40bb-9409-4d9f8cdbb168' class='xr-var-data-in' type='checkbox'><label for='data-7d7bab23-bc74-40bb-9409-4d9f8cdbb168' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'></dl></div><div class='xr-var-data'><table>
    <tr>
        <td>
            <table>
                <thead>
                    <tr>
                        <td> </td>
                        <th> Array </th>
                        <th> Chunk </th>
                    </tr>
                </thead>
                <tbody>

                    <tr>
                        <th> Bytes </th>
                        <td> 1.50 kiB </td>
                        <td> 1.50 kiB </td>
                    </tr>

                    <tr>
                        <th> Shape </th>
                        <td> (96, 2) </td>
                        <td> (96, 2) </td>
                    </tr>
                    <tr>
                        <th> Count </th>
                        <td> 2 Tasks </td>
                        <td> 1 Chunks </td>
                    </tr>
                    <tr>
                    <th> Type </th>
                    <td> float64 </td>
                    <td> numpy.ndarray </td>
                    </tr>
                </tbody>
            </table>
        </td>
        <td>
        <svg width="79" height="170" style="stroke:rgb(0,0,0);stroke-width:1" >

  <!-- Horizontal lines -->
  <line x1="0" y1="0" x2="29" y2="0" style="stroke-width:2" />
  <line x1="0" y1="120" x2="29" y2="120" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="0" y1="0" x2="0" y2="120" style="stroke-width:2" />
  <line x1="29" y1="0" x2="29" y2="120" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="0.0,0.0 29.2612335534925,0.0 29.2612335534925,120.0 0.0,120.0" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Text -->
  <text x="14.630617" y="140.000000" font-size="1.0rem" font-weight="100" text-anchor="middle" >2</text>
  <text x="49.261234" y="60.000000" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(0,49.261234,60.000000)">96</text>
</svg>
        </td>
    </tr>
</table></div></li><li class='xr-var-item'><div class='xr-var-name'><span class='xr-has-index'>lon</span></div><div class='xr-var-dims'>(lon)</div><div class='xr-var-dtype'>float64</div><div class='xr-var-preview xr-preview'>0.0 2.5 5.0 ... 352.5 355.0 357.5</div><input id='attrs-3e52667c-0376-48c3-b588-fd381557231b' class='xr-var-attrs-in' type='checkbox' ><label for='attrs-3e52667c-0376-48c3-b588-fd381557231b' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-6fea7cf4-8bb3-45b6-9017-b609c3ca18bf' class='xr-var-data-in' type='checkbox'><label for='data-6fea7cf4-8bb3-45b6-9017-b609c3ca18bf' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'><dt><span>axis :</span></dt><dd>X</dd><dt><span>bounds :</span></dt><dd>lon_bnds</dd><dt><span>long_name :</span></dt><dd>Longitude</dd><dt><span>standard_name :</span></dt><dd>longitude</dd><dt><span>units :</span></dt><dd>degrees_east</dd></dl></div><div class='xr-var-data'><pre>array([  0. ,   2.5,   5. ,   7.5,  10. ,  12.5,  15. ,  17.5,  20. ,  22.5,
        25. ,  27.5,  30. ,  32.5,  35. ,  37.5,  40. ,  42.5,  45. ,  47.5,
        50. ,  52.5,  55. ,  57.5,  60. ,  62.5,  65. ,  67.5,  70. ,  72.5,
        75. ,  77.5,  80. ,  82.5,  85. ,  87.5,  90. ,  92.5,  95. ,  97.5,
       100. , 102.5, 105. , 107.5, 110. , 112.5, 115. , 117.5, 120. , 122.5,
       125. , 127.5, 130. , 132.5, 135. , 137.5, 140. , 142.5, 145. , 147.5,
       150. , 152.5, 155. , 157.5, 160. , 162.5, 165. , 167.5, 170. , 172.5,
       175. , 177.5, 180. , 182.5, 185. , 187.5, 190. , 192.5, 195. , 197.5,
       200. , 202.5, 205. , 207.5, 210. , 212.5, 215. , 217.5, 220. , 222.5,
       225. , 227.5, 230. , 232.5, 235. , 237.5, 240. , 242.5, 245. , 247.5,
       250. , 252.5, 255. , 257.5, 260. , 262.5, 265. , 267.5, 270. , 272.5,
       275. , 277.5, 280. , 282.5, 285. , 287.5, 290. , 292.5, 295. , 297.5,
       300. , 302.5, 305. , 307.5, 310. , 312.5, 315. , 317.5, 320. , 322.5,
       325. , 327.5, 330. , 332.5, 335. , 337.5, 340. , 342.5, 345. , 347.5,
       350. , 352.5, 355. , 357.5])</pre></div></li><li class='xr-var-item'><div class='xr-var-name'><span>lon_bnds</span></div><div class='xr-var-dims'>(lon, bnds)</div><div class='xr-var-dtype'>float64</div><div class='xr-var-preview xr-preview'>dask.array&lt;chunksize=(144, 2), meta=np.ndarray&gt;</div><input id='attrs-53e7cfbc-65fc-41ee-a22f-125d48ed1df3' class='xr-var-attrs-in' type='checkbox' disabled><label for='attrs-53e7cfbc-65fc-41ee-a22f-125d48ed1df3' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-bfee2d8d-0197-49c3-a77c-2e09cea96628' class='xr-var-data-in' type='checkbox'><label for='data-bfee2d8d-0197-49c3-a77c-2e09cea96628' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'></dl></div><div class='xr-var-data'><table>
    <tr>
        <td>
            <table>
                <thead>
                    <tr>
                        <td> </td>
                        <th> Array </th>
                        <th> Chunk </th>
                    </tr>
                </thead>
                <tbody>

                    <tr>
                        <th> Bytes </th>
                        <td> 2.25 kiB </td>
                        <td> 2.25 kiB </td>
                    </tr>

                    <tr>
                        <th> Shape </th>
                        <td> (144, 2) </td>
                        <td> (144, 2) </td>
                    </tr>
                    <tr>
                        <th> Count </th>
                        <td> 2 Tasks </td>
                        <td> 1 Chunks </td>
                    </tr>
                    <tr>
                    <th> Type </th>
                    <td> float64 </td>
                    <td> numpy.ndarray </td>
                    </tr>
                </tbody>
            </table>
        </td>
        <td>
        <svg width="77" height="170" style="stroke:rgb(0,0,0);stroke-width:1" >

  <!-- Horizontal lines -->
  <line x1="0" y1="0" x2="27" y2="0" style="stroke-width:2" />
  <line x1="0" y1="120" x2="27" y2="120" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="0" y1="0" x2="0" y2="120" style="stroke-width:2" />
  <line x1="27" y1="0" x2="27" y2="120" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="0.0,0.0 27.054035963169408,0.0 27.054035963169408,120.0 0.0,120.0" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Text -->
  <text x="13.527018" y="140.000000" font-size="1.0rem" font-weight="100" text-anchor="middle" >2</text>
  <text x="47.054036" y="60.000000" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(-90,47.054036,60.000000)">144</text>
</svg>
        </td>
    </tr>
</table></div></li><li class='xr-var-item'><div class='xr-var-name'><span class='xr-has-index'>time</span></div><div class='xr-var-dims'>(time)</div><div class='xr-var-dtype'>object</div><div class='xr-var-preview xr-preview'>1850-01-16 12:00:00 ... 2014-12-...</div><input id='attrs-fa66e2b2-ef23-455a-8db4-665a2a5a5d75' class='xr-var-attrs-in' type='checkbox' ><label for='attrs-fa66e2b2-ef23-455a-8db4-665a2a5a5d75' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-fde8245a-faa3-4d7a-b37b-0f399df43362' class='xr-var-data-in' type='checkbox'><label for='data-fde8245a-faa3-4d7a-b37b-0f399df43362' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'><dt><span>axis :</span></dt><dd>T</dd><dt><span>bounds :</span></dt><dd>time_bnds</dd><dt><span>long_name :</span></dt><dd>time</dd><dt><span>standard_name :</span></dt><dd>time</dd></dl></div><div class='xr-var-data'><pre>array([cftime.DatetimeNoLeap(1850, 1, 16, 12, 0, 0, 0, has_year_zero=True),
       cftime.DatetimeNoLeap(1850, 2, 15, 0, 0, 0, 0, has_year_zero=True),
       cftime.DatetimeNoLeap(1850, 3, 16, 12, 0, 0, 0, has_year_zero=True),
       ...,
       cftime.DatetimeNoLeap(2014, 10, 16, 12, 0, 0, 0, has_year_zero=True),
       cftime.DatetimeNoLeap(2014, 11, 16, 0, 0, 0, 0, has_year_zero=True),
       cftime.DatetimeNoLeap(2014, 12, 16, 12, 0, 0, 0, has_year_zero=True)],
      dtype=object)</pre></div></li><li class='xr-var-item'><div class='xr-var-name'><span>time_bnds</span></div><div class='xr-var-dims'>(time, bnds)</div><div class='xr-var-dtype'>object</div><div class='xr-var-preview xr-preview'>dask.array&lt;chunksize=(1980, 2), meta=np.ndarray&gt;</div><input id='attrs-94f44221-a501-465a-b86f-d4603040f803' class='xr-var-attrs-in' type='checkbox' disabled><label for='attrs-94f44221-a501-465a-b86f-d4603040f803' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-5528b253-5fd8-439f-a2e3-0baa7b01bc51' class='xr-var-data-in' type='checkbox'><label for='data-5528b253-5fd8-439f-a2e3-0baa7b01bc51' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'></dl></div><div class='xr-var-data'><table>
    <tr>
        <td>
            <table>
                <thead>
                    <tr>
                        <td> </td>
                        <th> Array </th>
                        <th> Chunk </th>
                    </tr>
                </thead>
                <tbody>

                    <tr>
                        <th> Bytes </th>
                        <td> 30.94 kiB </td>
                        <td> 30.94 kiB </td>
                    </tr>

                    <tr>
                        <th> Shape </th>
                        <td> (1980, 2) </td>
                        <td> (1980, 2) </td>
                    </tr>
                    <tr>
                        <th> Count </th>
                        <td> 2 Tasks </td>
                        <td> 1 Chunks </td>
                    </tr>
                    <tr>
                    <th> Type </th>
                    <td> object </td>
                    <td> numpy.ndarray </td>
                    </tr>
                </tbody>
            </table>
        </td>
        <td>
        <svg width="75" height="170" style="stroke:rgb(0,0,0);stroke-width:1" >

  <!-- Horizontal lines -->
  <line x1="0" y1="0" x2="25" y2="0" style="stroke-width:2" />
  <line x1="0" y1="120" x2="25" y2="120" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="0" y1="0" x2="0" y2="120" style="stroke-width:2" />
  <line x1="25" y1="0" x2="25" y2="120" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="0.0,0.0 25.412616514582485,0.0 25.412616514582485,120.0 0.0,120.0" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Text -->
  <text x="12.706308" y="140.000000" font-size="1.0rem" font-weight="100" text-anchor="middle" >2</text>
  <text x="45.412617" y="60.000000" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(-90,45.412617,60.000000)">1980</text>
</svg>
        </td>
    </tr>
</table></div></li><li class='xr-var-item'><div class='xr-var-name'><span class='xr-has-index'>member_id</span></div><div class='xr-var-dims'>(member_id)</div><div class='xr-var-dtype'>&lt;U8</div><div class='xr-var-preview xr-preview'>&#x27;r1i1p1f1&#x27;</div><input id='attrs-0f69e78e-49ec-4afc-8f53-a295f2027b69' class='xr-var-attrs-in' type='checkbox' disabled><label for='attrs-0f69e78e-49ec-4afc-8f53-a295f2027b69' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-87c13bbe-daf8-4c36-a7fd-2715d074b054' class='xr-var-data-in' type='checkbox'><label for='data-87c13bbe-daf8-4c36-a7fd-2715d074b054' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'></dl></div><div class='xr-var-data'><pre>array([&#x27;r1i1p1f1&#x27;], dtype=&#x27;&lt;U8&#x27;)</pre></div></li></ul></div></li><li class='xr-section-item'><input id='section-56f73943-d6e7-4c54-a52c-55e445859337' class='xr-section-summary-in' type='checkbox'  checked><label for='section-56f73943-d6e7-4c54-a52c-55e445859337' class='xr-section-summary' >Data variables: <span>(1)</span></label><div class='xr-section-inline-details'></div><div class='xr-section-details'><ul class='xr-var-list'><li class='xr-var-item'><div class='xr-var-name'><span>tas</span></div><div class='xr-var-dims'>(member_id, time, lat, lon)</div><div class='xr-var-dtype'>float32</div><div class='xr-var-preview xr-preview'>dask.array&lt;chunksize=(1, 990, 96, 144), meta=np.ndarray&gt;</div><input id='attrs-b3d85ab9-5074-4b36-9944-931382059c32' class='xr-var-attrs-in' type='checkbox' ><label for='attrs-b3d85ab9-5074-4b36-9944-931382059c32' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-aed941a4-523f-4fc8-b694-8be2d39dc0ec' class='xr-var-data-in' type='checkbox'><label for='data-aed941a4-523f-4fc8-b694-8be2d39dc0ec' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'><dt><span>cell_measures :</span></dt><dd>area: areacella</dd><dt><span>cell_methods :</span></dt><dd>area: time: mean</dd><dt><span>comment :</span></dt><dd>near-surface (usually, 2 meter) air temperature</dd><dt><span>history :</span></dt><dd>2019-08-15T12:42:20Z altered by CMOR: Treated scalar dimension: &#x27;height&#x27;. 2019-08-15T12:42:21Z altered by CMOR: Converted type from &#x27;d&#x27; to &#x27;f&#x27;.</dd><dt><span>long_name :</span></dt><dd>Near-Surface Air Temperature</dd><dt><span>original_name :</span></dt><dd>TREFHT</dd><dt><span>standard_name :</span></dt><dd>air_temperature</dd><dt><span>units :</span></dt><dd>K</dd></dl></div><div class='xr-var-data'><table>
    <tr>
        <td>
            <table>
                <thead>
                    <tr>
                        <td> </td>
                        <th> Array </th>
                        <th> Chunk </th>
                    </tr>
                </thead>
                <tbody>

                    <tr>
                        <th> Bytes </th>
                        <td> 104.41 MiB </td>
                        <td> 52.21 MiB </td>
                    </tr>

                    <tr>
                        <th> Shape </th>
                        <td> (1, 1980, 96, 144) </td>
                        <td> (1, 990, 96, 144) </td>
                    </tr>
                    <tr>
                        <th> Count </th>
                        <td> 5 Tasks </td>
                        <td> 2 Chunks </td>
                    </tr>
                    <tr>
                    <th> Type </th>
                    <td> float32 </td>
                    <td> numpy.ndarray </td>
                    </tr>
                </tbody>
            </table>
        </td>
        <td>
        <svg width="347" height="154" style="stroke:rgb(0,0,0);stroke-width:1" >

  <!-- Horizontal lines -->
  <line x1="0" y1="0" x2="25" y2="0" style="stroke-width:2" />
  <line x1="0" y1="25" x2="25" y2="25" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="0" y1="0" x2="0" y2="25" style="stroke-width:2" />
  <line x1="25" y1="0" x2="25" y2="25" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="0.0,0.0 25.412616514582485,0.0 25.412616514582485,25.412616514582485 0.0,25.412616514582485" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Text -->
  <text x="12.706308" y="45.412617" font-size="1.0rem" font-weight="100" text-anchor="middle" >1</text>
  <text x="45.412617" y="12.706308" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(0,45.412617,12.706308)">1</text>


  <!-- Horizontal lines -->
  <line x1="95" y1="0" x2="165" y2="70" style="stroke-width:2" />
  <line x1="95" y1="34" x2="165" y2="104" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="95" y1="0" x2="95" y2="34" style="stroke-width:2" />
  <line x1="130" y1="35" x2="130" y2="69" />
  <line x1="165" y1="70" x2="165" y2="104" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="95.0,0.0 165.58823529411765,70.58823529411765 165.58823529411765,104.9007638365262 95.0,34.31252854240854" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="95" y1="0" x2="131" y2="0" style="stroke-width:2" />
  <line x1="130" y1="35" x2="167" y2="35" />
  <line x1="165" y1="70" x2="202" y2="70" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="95" y1="0" x2="165" y2="70" style="stroke-width:2" />
  <line x1="131" y1="0" x2="202" y2="70" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="95.0,0.0 131.7664046331794,0.0 202.35463992729706,70.58823529411765 165.58823529411765,70.58823529411765" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="165" y1="70" x2="202" y2="70" style="stroke-width:2" />
  <line x1="165" y1="104" x2="202" y2="104" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="165" y1="70" x2="165" y2="104" style="stroke-width:2" />
  <line x1="202" y1="70" x2="202" y2="104" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="165.58823529411765,70.58823529411765 202.35463992729706,70.58823529411765 202.35463992729706,104.9007638365262 165.58823529411765,104.9007638365262" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Text -->
  <text x="183.971438" y="124.900764" font-size="1.0rem" font-weight="100" text-anchor="middle" >144</text>
  <text x="222.354640" y="87.744500" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(0,222.354640,87.744500)">96</text>
  <text x="120.294118" y="89.606646" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(45,120.294118,89.606646)">1980</text>
</svg>
        </td>
    </tr>
</table></div></li></ul></div></li><li class='xr-section-item'><input id='section-48a8c101-2477-40f5-939c-a0c4e232b414' class='xr-section-summary-in' type='checkbox'  ><label for='section-48a8c101-2477-40f5-939c-a0c4e232b414' class='xr-section-summary' >Attributes: <span>(54)</span></label><div class='xr-section-inline-details'></div><div class='xr-section-details'><dl class='xr-attrs'><dt><span>Conventions :</span></dt><dd>CF-1.7 CMIP-6.2</dd><dt><span>activity_id :</span></dt><dd>CMIP</dd><dt><span>branch_method :</span></dt><dd>Hybrid-restart from year 1600-01-01 of piControl</dd><dt><span>branch_time :</span></dt><dd>0.0</dd><dt><span>branch_time_in_child :</span></dt><dd>0.0</dd><dt><span>branch_time_in_parent :</span></dt><dd>430335.0</dd><dt><span>cmor_version :</span></dt><dd>3.5.0</dd><dt><span>contact :</span></dt><dd>Please send any requests or bug reports to noresm-ncc@met.no.</dd><dt><span>creation_date :</span></dt><dd>2019-08-15T12:42:21Z</dd><dt><span>data_specs_version :</span></dt><dd>01.00.31</dd><dt><span>experiment :</span></dt><dd>all-forcing simulation of the recent past</dd><dt><span>experiment_id :</span></dt><dd>historical</dd><dt><span>external_variables :</span></dt><dd>areacella</dd><dt><span>forcing_index :</span></dt><dd>1</dd><dt><span>frequency :</span></dt><dd>mon</dd><dt><span>further_info_url :</span></dt><dd>https://furtherinfo.es-doc.org/CMIP6.NCC.NorESM2-LM.historical.none.r1i1p1f1</dd><dt><span>grid :</span></dt><dd>finite-volume grid with 1.9x2.5 degree lat/lon resolution</dd><dt><span>grid_label :</span></dt><dd>gn</dd><dt><span>history :</span></dt><dd>2019-08-15T12:42:21Z ; CMOR rewrote data to be consistent with CMIP6, CF-1.7 CMIP-6.2 and CF standards.</dd><dt><span>initialization_index :</span></dt><dd>1</dd><dt><span>institution :</span></dt><dd>NorESM Climate modeling Consortium consisting of CICERO (Center for International Climate and Environmental Research, Oslo 0349), MET-Norway (Norwegian Meteorological Institute, Oslo 0313), NERSC (Nansen Environmental and Remote Sensing Center, Bergen 5006), NILU (Norwegian Institute for Air Research, Kjeller 2027), UiB (University of Bergen, Bergen 5007), UiO (University of Oslo, Oslo 0313) and UNI (Uni Research, Bergen 5008), Norway. Mailing address: NCC, c/o MET-Norway, Henrik Mohns plass 1, Oslo 0313, Norway</dd><dt><span>institution_id :</span></dt><dd>NCC</dd><dt><span>license :</span></dt><dd>CMIP6 model data produced by NCC is licensed under a Creative Commons Attribution ShareAlike 4.0 International License (https://creativecommons.org/licenses). Consult https://pcmdi.llnl.gov/CMIP6/TermsOfUse for terms of use governing CMIP6 output, including citation requirements and proper acknowledgment. Further information about this data, including some limitations, can be found via the further_info_url (recorded as a global attribute in this file) and at https:///pcmdi.llnl.gov/. The data producers and data providers make no warranty, either express or implied, including, but not limited to, warranties of merchantability and fitness for a particular purpose. All liabilities arising from the supply of the information (including any liability arising in negligence) are excluded to the fullest extent permitted by law.</dd><dt><span>mip_era :</span></dt><dd>CMIP6</dd><dt><span>model_id :</span></dt><dd>NorESM2-LM</dd><dt><span>nominal_resolution :</span></dt><dd>250 km</dd><dt><span>parent_activity_id :</span></dt><dd>CMIP</dd><dt><span>parent_experiment_id :</span></dt><dd>piControl</dd><dt><span>parent_mip_era :</span></dt><dd>CMIP6</dd><dt><span>parent_source_id :</span></dt><dd>NorESM2-LM</dd><dt><span>parent_sub_experiment_id :</span></dt><dd>none</dd><dt><span>parent_time_units :</span></dt><dd>days since 0421-01-01</dd><dt><span>parent_variant_label :</span></dt><dd>r1i1p1f1</dd><dt><span>physics_index :</span></dt><dd>1</dd><dt><span>product :</span></dt><dd>model-output</dd><dt><span>realization_index :</span></dt><dd>1</dd><dt><span>realm :</span></dt><dd>atmos</dd><dt><span>run_variant :</span></dt><dd>N/A</dd><dt><span>source :</span></dt><dd>NorESM2-LM (2017): 
aerosol: OsloAero
atmos: CAM-OSLO (2 degree resolution; 144 x 96; 32 levels; top level 3 mb)
atmosChem: OsloChemSimp
land: CLM
landIce: CISM
ocean: MICOM (1 degree resolution; 360 x 384; 70 levels; top grid cell minimum 0-2.5 m [native model uses hybrid density and generic upper-layer coordinate interpolated to z-level for contributed data])
ocnBgchem: HAMOCC
seaIce: CICE</dd><dt><span>source_id :</span></dt><dd>NorESM2-LM</dd><dt><span>source_type :</span></dt><dd>AOGCM</dd><dt><span>status :</span></dt><dd>2020-04-30;created; by gcs.cmip6.ldeo@gmail.com</dd><dt><span>sub_experiment :</span></dt><dd>none</dd><dt><span>sub_experiment_id :</span></dt><dd>none</dd><dt><span>table_id :</span></dt><dd>Amon</dd><dt><span>table_info :</span></dt><dd>Creation Date:(24 July 2019) MD5:08e314340b9dd440c36c1b8e484514f4</dd><dt><span>title :</span></dt><dd>NorESM2-LM output prepared for CMIP6</dd><dt><span>tracking_id :</span></dt><dd>hdl:21.14100/2486cf87-033c-4848-ab3e-e828c3b7c759
hdl:21.14100/8d2ec216-30d3-4f90-8664-7cf75e29780b
hdl:21.14100/9f798305-d0cf-4eac-89b0-b13dc25574ca
hdl:21.14100/e0da9277-4784-446b-b3b5-ba33bb18cc3a
hdl:21.14100/ca48487c-955a-4b15-9582-5ce55ce39c73
hdl:21.14100/fc7602aa-fd09-49d8-94c6-e619622c211b
hdl:21.14100/cd51a9c2-0a94-45b2-8d9b-59bcb3467214
hdl:21.14100/9b7ad334-1217-4229-98e8-b5a9287c0e5e
hdl:21.14100/043ca527-40c7-451f-990b-ccc94720f612
hdl:21.14100/a9613498-ea0d-4be4-97e8-77661872798d
hdl:21.14100/52699832-d86c-491b-a0a4-2f27d9ee4d69
hdl:21.14100/358c4a18-33b2-4c44-bd30-f6125a4df63c
hdl:21.14100/77254eaf-90ca-44e7-9a3c-763a840e188f
hdl:21.14100/62aed49c-6ca5-4027-80fe-4ddd99af575a
hdl:21.14100/83ffe172-769b-42b1-b979-b92d768f60fd
hdl:21.14100/91a8c4e2-d33b-4b87-a978-0247d8112ef3
hdl:21.14100/e8fb384f-d5a7-41e1-bf12-b8de3e2ffb80</dd><dt><span>variable_id :</span></dt><dd>tas</dd><dt><span>variant_label :</span></dt><dd>r1i1p1f1</dd><dt><span>netcdf_tracking_ids :</span></dt><dd>hdl:21.14100/2486cf87-033c-4848-ab3e-e828c3b7c759
hdl:21.14100/8d2ec216-30d3-4f90-8664-7cf75e29780b
hdl:21.14100/9f798305-d0cf-4eac-89b0-b13dc25574ca
hdl:21.14100/e0da9277-4784-446b-b3b5-ba33bb18cc3a
hdl:21.14100/ca48487c-955a-4b15-9582-5ce55ce39c73
hdl:21.14100/fc7602aa-fd09-49d8-94c6-e619622c211b
hdl:21.14100/cd51a9c2-0a94-45b2-8d9b-59bcb3467214
hdl:21.14100/9b7ad334-1217-4229-98e8-b5a9287c0e5e
hdl:21.14100/043ca527-40c7-451f-990b-ccc94720f612
hdl:21.14100/a9613498-ea0d-4be4-97e8-77661872798d
hdl:21.14100/52699832-d86c-491b-a0a4-2f27d9ee4d69
hdl:21.14100/358c4a18-33b2-4c44-bd30-f6125a4df63c
hdl:21.14100/77254eaf-90ca-44e7-9a3c-763a840e188f
hdl:21.14100/62aed49c-6ca5-4027-80fe-4ddd99af575a
hdl:21.14100/83ffe172-769b-42b1-b979-b92d768f60fd
hdl:21.14100/91a8c4e2-d33b-4b87-a978-0247d8112ef3
hdl:21.14100/e8fb384f-d5a7-41e1-bf12-b8de3e2ffb80</dd><dt><span>version_id :</span></dt><dd>v20190815</dd><dt><span>intake_esm_varname :</span></dt><dd>[&#x27;tas&#x27;]</dd><dt><span>intake_esm_dataset_key :</span></dt><dd>CMIP.NCC.NorESM2-LM.historical.Amon.gn</dd></dl></div></li></ul></div></div>



Plot the first timestep


```python
projection = ccrs.Mercator(central_longitude=-10)

f, ax = plt.subplots(subplot_kw=dict(projection=projection))

dset['tas'].isel(time=0).plot(transform=ccrs.PlateCarree(), cbar_kwargs=dict(shrink=0.7), cmap='coolwarm')
ax.coastlines()
```




    <cartopy.mpl.feature_artist.FeatureArtist at 0x7f1212118670>




    
![png](index_files/index_13_1.png)
    


## Compute weighted mean

1. Creating weights: for a rectangular grid the cosine of the latitude is proportional to the grid cell area.
2. Compute weighted mean values


```python
def computeWeightedMean(ds):
    # Compute weights based on the xarray you pass
    weights = np.cos(np.deg2rad(ds.lat))
    weights.name = "weights"
    # Compute weighted mean
    air_weighted = ds.weighted(weights)
    weighted_mean = air_weighted.mean(("lon", "lat"))
    return weighted_mean
```

## Compute weighted average over the entire globe


```python
weighted_mean = computeWeightedMean(dset)
```

## Comparison with unweighted mean
- We select a time range
- Note how the weighted mean temperature is higher than the unweighted.


```python
weighted_mean['tas'].sel(time=slice('2000-01-01', '2010-01-01')).plot(label="weighted")
dset['tas'].sel(time=slice('2000-01-01', '2010-01-01')).mean(("lon", "lat")).plot(label="unweighted")

plt.legend()
```




    <matplotlib.legend.Legend at 0x7f12138a66b0>




    
![png](index_files/index_19_1.png)
    


## Compute Weigted arctic average
Let's try to also take only the data above 60$^\circ$


```python
weighted_mean = computeWeightedMean(dset.where(dset['lat']>60.))
```


```python
weighted_mean['tas'].sel(time=slice('2000-01-01', '2010-01-01')).plot(label="weighted")
dset['tas'].where(dset['lat']>60.).sel(time=slice('2000-01-01', '2010-01-01')).mean(("lon", "lat")).plot(label="unweighted")

plt.legend()
```




    <matplotlib.legend.Legend at 0x7f1211b59480>




    
![png](index_files/index_22_1.png)
    



```python

```


```python

```
