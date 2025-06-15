# SUMMARY

This dataset provides global 10K most populous locations from Sentinel1/2 and Landsat missions. The dataset was collated from [SSL4EO-L](https://huggingface.co/datasets/torchgeo/ssl4eo_l) and [SSL4EO-S12](https://huggingface.co/datasets/wangyi111/SSL4EO-S12). Apart from original metadata from the geotifs, it contains an additional context file which contains information extracted from filenames and geotif metadata for ease of access. Data has been preprocessed save Normalization for quick access when developing ML products.


ssl4eo-s12-landsat-combined

## About
This dataset combines SSL4EO-L and SSL4EO-S12 to form a combined multiview dataset. The combined dataset has 66800 locations with view from atleast 5 sensors.
The sensors available are:

```bash
- s1_grd
- s2_l1c
- s2_l2a
- ssl4eo_l_oli_sr
- ssl4eo_l_tirs_toa
- ssl4eo_l_etm_sr
- ssl4eo_l_etm_toa
- ssl4eo_l_tm_toa
```

The data has been very kindly hosted in AWS Open Data Registry, in ```ssl4eo-s12-landsat-combined``` bucket. 


## Data
Each of the above folders has atleast 8 files and at max 12 files. All folders have original image data as .h5 format, along with context for four seasons. In most cases, there might be additional metadata file which contains details about the original satellite metadata. The file details are as given below:

```bash
- <datestr>.h5 -> contains all bands fom the relevant satellite and product 
- context_<datestr>.json -> contains releavant context for the corresponding datestr.h5
- meta_<datestr>.json -> Additional metadata if available for the corresponding datestr.h5
```

All input files are of shape 224, 224. Additionally, Lee-filtering based speckle fltering has been done for Sentinel1 products have been and Sentinel2 products have been scaled according to information provided [here](https://forum.sentinel-hub.com/t/normalization-of-sentinel-data-for-ml-downstream/5459/2). Sentinel data has additionally been scaled to be between 0 and 255. Landsat data is already in between that range. No normalization has been applied to the input data.

## Data Access
Details for AWS access can be found [here](https://registry.opendata.aws/ssl4eo-multi-product-data/) using **aws s3 ls --no-sign-request s3://ssl4eo-s12-landsat-combined/**.

To access the above via cli, ```awscli``` needs to be installed on the system:

```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
./aws/install -i ~/local/aws-cli -b ~/bin/aws-cli
vi ~/.bashrc
export PATH="<existing_path_extry>:$HOME/bin/aws-cli/:$PATH"
# save ~/.bashrc
source ~/.bashrc
aws --version
```

After above, ```aws s3 ls --no-sign-request s3://ssl4eo-s12-landsat-combined/``` can be used to view to folders in the bucket. However, I would recommend using the tutorial to access the data in the bucket, because of the huge list of locations.

To view a single location, try one of the locations present in ```location_prefix.txt``` with the above command. For example

```bash
aws s3 ls --no-sign-request s3://ssl4eo-s12-landsat-combined/0078423/
```
This will output the modalities present for the selected location.

```bash
PRE s1_grd/
PRE s2_l1c/
PRE s2_l2a/
PRE ssl4eo_l_etm_sr/
PRE ssl4eo_l_etm_toa/
PRE ssl4eo_l_oli_sr/
PRE ssl4eo_l_tirs_toa/
PRE ssl4eo_l_tm_toa/
```

Each modality has atleast 8 files: Season wise ```.h5``` data as well dictionary based ```context_julian_date.json``` 
containing lat, lon, date, season and year of access. In some cases, we have additional meta files containing satellite metadata.

## Data Tutorial

```bash
from constants import Ssl4eoMulti

files = Ssl4eoMulti.list_all_files_recursively(prefix="0000001")
files
[
  '0000001/s1_grd/20201101.h5',
  '0000001/s1_grd/20210205.h5',
  '0000001/s1_grd/20210430.h5',
  '0000001/s1_grd/20210804.h5',
  '0000001/s1_grd/context_20201101.json',
  '0000001/s1_grd/context_20210205.json',
  '0000001/s1_grd/context_20210430.json',
  '0000001/s1_grd/context_20210804.json',
  '0000001/s1_grd/meta_20201101.json',
  '0000001/s1_grd/meta_20210205.json',
  '0000001/s1_grd/meta_20210430.json',
  '0000001/s1_grd/meta_20210804.json',
  '0000001/s2_l1c/20201104.h5',
  '0000001/s2_l1c/20210123.h5',
  '0000001/s2_l1c/20210508.h5',
  '0000001/s2_l1c/20210811.h5',
  ...
  '0000001/ssl4eo_l_oli_sr/20210306.h5',
  '0000001/ssl4eo_l_oli_sr/20210829.h5',
  '0000001/ssl4eo_l_oli_sr/20211203.h5',
  '0000001/ssl4eo_l_oli_sr/20220528.h5',
  '0000001/ssl4eo_l_oli_sr/context_20210306.json',
  '0000001/ssl4eo_l_oli_sr/context_20210829.json',
  '0000001/ssl4eo_l_oli_sr/context_20211203.json',
  '0000001/ssl4eo_l_oli_sr/context_20220528.json',
  '0000001/ssl4eo_l_oli_sr/meta_20210306.json',
  '0000001/ssl4eo_l_oli_sr/meta_20210829.json',
  '0000001/ssl4eo_l_oli_sr/meta_20211203.json',
  '0000001/ssl4eo_l_oli_sr/meta_20220528.json'
]

files[0]
'0000001/s1_grd/20201101.h5'

data = Ssl4eoMulti.fetch_and_process_file(files[1])
relevant_ctx_file = files[1].replace(os.path.split(files[1])[1], f"context_{os.path.split(files[1])[1].split('.')[0]}.json").strip()

ctx = Ssl4eoMulti.fetch_and_process_file(relevant_ctx_file)
ctx
{
  'year': '2021',
  'season': 'summer',
  'longitude': -55.803693900822836,
  'latitude': -20.446156247997944,
  'datestr': '20210205',
  'date_format': '%Y%m%d'
}
```


## License

[Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International (CC BY-NC-SA 4.0)](https://creativecommons.org/licenses/by-nc-sa/4.0/)

## Citations

```
@article{wang2022ssl4eo,
  title={SSL4EO-S12: A Large-Scale Multi-Modal, Multi-Temporal Dataset for Self-Supervised Learning in Earth Observation},
  author={Wang, Yi and Braham, Nassim Ait Ali and Xiong, Zhitong and Liu, Chenying and Albrecht, Conrad M and Zhu, Xiao Xiang},
  journal={arXiv preprint arXiv:2211.07044},
  year={2022}
}

@misc{stewart2023ssl4eol,
    title={SSL4EO-L: Datasets and Foundation Models for Landsat Imagery}, 
    author={Adam J. Stewart and Nils Lehmann and Isaac A. Corley and Yi Wang and Yi-Chia Chang and Nassim Ait Ali Braham and Shradha Sehgal and Caleb Robinson and Arindam Banerjee},
    year={2023},
    eprint={2306.09424},
    archivePrefix={arXiv},
    primaryClass={cs.LG}
}
```

## Acknowledgement
This work was completed entirely using the Edinburgh International Data Facility (EIDF) and the Data-Driven Innovation Programme at the University of Edinburgh. 
