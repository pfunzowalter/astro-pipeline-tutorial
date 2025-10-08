# Process MeerKat Tutorial
This tutorial is designed to help you use the processMeerKAT Pipeline on the ilifu cluster, utilizing public datasets from the SARAO archive. Details about the processMeerKAT pipeline can be found in the official [documentation](https://idia-pipelines.github.io/docs/processMeerKAT). We highly recommend reviewing these resources to gain a deeper understanding of the pipeline’s features and usage. The documentation is accompanied by a detailed [tutorial](https://idia-pipelines.github.io/docs/processMeerKAT/deep-2-tutorial/), which is invaluable for learning how to effectively process MeerKAT data.

In our case, however, we focus on a different dataset that presents some specific challenges, such as missing **FIELDS** and **reference antenna** in the config file.

## Datasets
There are two public datasets available on ilifu at `/idia/data/public`: `1491550051` and `1525469431`.

- `1491550051` is a Measurement set (MS) featured in a processMeerKAT tutorial [here](https://idia-pipelines.github.io/docs/processMeerKAT/deep-2-tutorial/). Please refer to this resource for detailed information about the data. Note that the tutorial is based on **version 1.0**, while currently the pipeline is at version 1.1.
- `1525469431` is a MeerKAT observation of the massive galaxy cluster ACT-CL J2023.3-5535 in the L-Band. The name indicates its discovery by the Atacama Cosmology Telescope (ACT), with "CL" denoting "Cluster" and "J2023.3-5535" representing its sky coordinates (Right Ascension 20h 23m 18s, Declination -55° 35' 00"). For more background on this cluster, see [this paper](https://academic.oup.com/mnras/advance-article/doi/10.1093/mnras/staf1499/8250033).

Below is an image of this target in the UHF band:

![ACT-CL J2023.3-5535 in UHF band](1589920440_continuum_image_J2023_3-5535_IClean.png)

Image source: [SARAO SDP+Pipeline+overview](https://skaafrica.atlassian.net/wiki/spaces/ESDKB/pages/338723406/SDP+pipelines+overview)

## Observation details
Before starting the calibration process, it is recommended -if not necessary- to get some basic information about the data set. General information about the MeerKat observations can be accessed from the SARAO archive but here you would like to check some basic infomation about the dataset such as the antenna's, Correlations and Fieds, that are available. This is crucial because the processMeerKAT pipeline configuration uses the M059 antenna as the reference by default. If M059 is not present in your dataset, you will need to select a different reference antenna. You would also need to know the Target, and Calibrator and double check with the processMeerKAT if all is good. For Some reasons mentinoned in the upcoming tutorial you may have to manually update the pipelines config file.

## Lets InSpect the data with CASA interactively on ilifu
1. Log in to slurm-ilifu:  
   ```bash
   ssh walter@slurm.ilifu.ac.za
   ```
2. Start an interactive session (the Devel partition is sufficient):
    ```bash
    sinteractive
    ```
3. Navigate to your workspace for processing. Ideally, use `/scratch3/user` or `/scratch3/projects`. Then launch a CASA-stable container:
    ```bash
    singularity shell /idia/software/containers/casa-stable-v6.7.0-31-py3.10-2025-04-08.sif
    ```
4. Start CASA and use the listobs() task to inspect your data. This task provides a summary of scans, frequency setup, source list, and antenna locations.
In the container shell, initialize CASA with:
    ```bash
    casa
    ```
 - **If CASA fails to start due to missing data, create the required directory and restart CASA**
    ```bash
    mkdir ~/.casa/data
    casa
    ```
 - Initialize the listobs() task by running:
    ```bash
    inp listobs
    ```
    This will open the task interface (Shown below) for further configuration.

    ```
    Listobs -- Summary of a MeasurementSet and list it in the logger or in a file
    vis            = ''                      # Name of input visibility file (MS)
    selectdata     = True                    # Data selection parameters
    spw         = ''                      # Selection based on
                                            # spectral-window/frequency/channel.
    field       = ''                      # Selection based on field names or field
                                            # index numbers. Default is all.
    antenna     = ''                      # Selection based on antenna/baselines.
                                            # Default is all.
    uvrange     = ''                      # Selection based on uv range. Default: entire
                                            # range. Default units: meters.
    timerange   = ''                      # Selection based on time range. Default is
                                            # entire range.
    correlation = ''                      # Selection based on correlation. Default is
                                            # all.
    scan        = ''                      # Selection based on scan numbers. Default is
                                            # all.
    intent      = ''                      # Selection based on observation intent.
                                            # Default is all.
    feed        = ''                      # Selection based on multi-feed numbers: Not
                                            # yet implemented
    array       = ''                      # Selection based on (sub)array numbers.
                                            # Default is all.
    observation = ''                      # Selection based on observation ID. Default
                                            # is all.
    verbose        = True                    # Controls level of information detail
                                            # reported. True reports more than False.
    listfile       = ''                      # Name of disk file to write output. Default
                                            # is none (output is written to logger only).
    listunfl       = False                   # List unflagged row counts? If true, it can
                                            # have significant negative performance
                                            # impact.
    cachesize      = 50.0                    # EXPERIMENTAL. Maximum size in megabytes of
                                            # cache in which data structures can be held.

    ```

    Then point the task to the correct MS: 
    ```bash
    vis = /scratch3/users/walter/pipeline-training/1525469431/1525469431_sdp_l0_2025-09-03T09-10-19_s1f.ms
    ``` 
    and also 

    ```bash
    listfile = details_1525469431.txt
    ```

    - Once all that is setup we can run listobs task with command `go`

    We can now inspect details about our observation in the `details_1525469431.txt` which should be similar to the output below.
    ```
    ================================================================================
            MeasurementSet Name:  /scratch3/users/walter/pipeline-training/1525469431/1525469431_sdp_l0_2025-09-03T09-10-19_s1f.ms      MS Version 2
    ================================================================================
    Observer: Tony     Project: 20180504-0015
    Observation: MeerKAT
    Data records: 630840       Total elapsed time = 21606.9 seconds
    Observed from   04-May-2018/21:31:15.6   to   05-May-2018/03:31:22.5 (UTC)

    ObservationID = 0         ArrayID = 0
    Date        Timerange (UTC)          Scan  FldId FieldName             nRows     SpwIds   Average Interval(s)    ScanIntent
    04-May-2018/21:31:15.6 - 21:51:11.1     1      0 ACT-CLJ2023.3-5535       35880  [0]  [4] [TARGET]
                21:51:31.1 - 21:53:27.1     2      1 J1939-6342                3480  [0]  [4] [CALIBRATE_AMPLI,CALIBRATE_PHASE]
                21:53:47.1 - 22:13:42.6     3      0 ACT-CLJ2023.3-5535       35880  [0]  [4] [TARGET]
                22:14:02.6 - 22:16:02.5     4      1 J1939-6342                3600  [0]  [4] [CALIBRATE_AMPLI,CALIBRATE_PHASE]
                22:16:22.5 - 22:36:18.0     5      0 ACT-CLJ2023.3-5535       35880  [0]  [4] [TARGET]
                22:36:38.0 - 22:38:33.9     6      1 J1939-6342                3480  [0]  [4] [CALIBRATE_AMPLI,CALIBRATE_PHASE]
                22:38:53.9 - 22:58:49.4     7      0 ACT-CLJ2023.3-5535       35880  [0]  [4] [TARGET]
                22:59:05.4 - 23:01:05.4     8      1 J1939-6342                3600  [0]  [4] [CALIBRATE_AMPLI,CALIBRATE_PHASE]
                23:01:21.4 - 23:21:20.8     9      0 ACT-CLJ2023.3-5535       36000  [0]  [4] [TARGET]
                23:21:36.8 - 23:23:32.8    10      1 J1939-6342                3480  [0]  [4] [CALIBRATE_AMPLI,CALIBRATE_PHASE]
                23:23:52.8 - 23:43:48.3    11      0 ACT-CLJ2023.3-5535       35880  [0]  [4] [TARGET]
                23:44:04.3 - 23:46:00.2    12      1 J1939-6342                3480  [0]  [4] [CALIBRATE_AMPLI,CALIBRATE_PHASE]
                23:46:20.2 - 00:06:15.7    13      0 ACT-CLJ2023.3-5535       35880  [0]  [4] [TARGET]
    05-May-2018/00:06:35.7 - 00:08:31.6    14      1 J1939-6342                3480  [0]  [4] [CALIBRATE_AMPLI,CALIBRATE_PHASE]
                00:08:47.6 - 00:28:47.1    15      0 ACT-CLJ2023.3-5535       36000  [0]  [4] [TARGET]
                00:29:03.1 - 00:31:03.1    16      1 J1939-6342                3600  [0]  [4] [CALIBRATE_AMPLI,CALIBRATE_PHASE]
                00:31:19.1 - 00:51:14.6    17      0 ACT-CLJ2023.3-5535       35880  [0]  [4] [TARGET]
                00:51:34.6 - 00:53:30.5    18      1 J1939-6342                3480  [0]  [4] [CALIBRATE_AMPLI,CALIBRATE_PHASE]
                00:53:50.5 - 01:13:46.0    19      0 ACT-CLJ2023.3-5535       35880  [0]  [4] [TARGET]
                01:14:06.0 - 01:16:05.9    20      1 J1939-6342                3600  [0]  [4] [CALIBRATE_AMPLI,CALIBRATE_PHASE]
                01:16:21.9 - 01:36:21.4    21      0 ACT-CLJ2023.3-5535       36000  [0]  [4] [TARGET]
                01:36:41.4 - 01:38:37.4    22      1 J1939-6342                3480  [0]  [4] [CALIBRATE_AMPLI,CALIBRATE_PHASE]
                01:38:57.4 - 01:58:52.8    23      0 ACT-CLJ2023.3-5535       35880  [0]  [4] [TARGET]
                01:59:12.8 - 02:01:08.8    24      1 J1939-6342                3480  [0]  [4] [CALIBRATE_AMPLI,CALIBRATE_PHASE]
                02:01:28.8 - 02:21:28.3    25      0 ACT-CLJ2023.3-5535       36000  [0]  [4] [TARGET]
                02:21:48.3 - 02:23:44.2    26      1 J1939-6342                3480  [0]  [4] [CALIBRATE_AMPLI,CALIBRATE_PHASE]
                02:24:04.2 - 02:43:59.7    27      0 ACT-CLJ2023.3-5535       35880  [0]  [4] [TARGET]
                02:44:19.7 - 02:46:19.6    28      1 J1939-6342                3600  [0]  [4] [CALIBRATE_AMPLI,CALIBRATE_PHASE]
                02:46:39.6 - 03:06:35.1    29      0 ACT-CLJ2023.3-5535       35880  [0]  [4] [TARGET]
                03:06:55.1 - 03:08:51.1    30      1 J1939-6342                3480  [0]  [4] [CALIBRATE_AMPLI,CALIBRATE_PHASE]
                03:09:11.1 - 03:29:06.6    31      0 ACT-CLJ2023.3-5535       35880  [0]  [4] [TARGET]
                03:29:26.5 - 03:31:22.5    32      1 J1939-6342                3480  [0]  [4] [CALIBRATE_AMPLI,CALIBRATE_PHASE]
            (nRows = Total number of rows per scan)
    Fields: 2
    ID   Code Name                RA               Decl           Epoch   SrcId      nRows
    0    T    ACT-CLJ2023.3-5535  20:23:18.820000 -55.35.19.00000 J2000   0         574560
    1    T    J1939-6342          19:39:25.050000 -63.42.43.60000 J2000   1          56280
    Spectral Windows:  (1 unique spectral windows and 1 unique polarization setups)
    SpwID  Name   #Chans   Frame   Ch0(MHz)  ChanWid(kHz)  TotBW(kHz) CtrFreq(MHz)  Corrs
    0      none    4096   TOPO     856.000       208.984    856000.0   1283.8955   XX  YY
    Sources: 2
    ID   Name                SpwId RestFreq(MHz)
    0    ACT-CLJ2023.3-5535  0     -
    1    J1939-6342          0     -
    NB: No systemic velocity information found in SOURCE table.
    Antennas: 16:
    ID   Name  Station   Diam.    Long.         Lat.                Offset from array center (m)                ITRF Geocentric coordinates (m)
                                                                        East         North     Elevation               x               y               z
    0    m001  m001      13.5 m   +021.26.38.0  -30.32.38.1          0.0615       66.0615      -13.7427  5109237.630487  2006805.679168 -3239069.988673
    1    m002  m002      13.5 m   +021.26.36.8  -30.32.39.8        -33.1730       13.5856      -13.7080  5109224.986208  2006765.006527 -3239115.200439
    2    m003  m003      13.5 m   +021.26.35.5  -30.32.39.1        -67.5799       35.5468      -14.0038  5109247.715781  2006736.968312 -3239096.136391
    3    m006  m006      13.5 m   +021.26.37.3  -30.32.42.1        -19.2906      -57.6078      -13.3758  5109186.500724  2006764.805186 -3239176.683299
    4    m008  m008      13.5 m   +021.26.34.5  -30.32.49.9        -94.5903     -297.2076      -12.8094  5109101.139483  2006650.380018 -3239383.318912
    5    m015  m015      13.5 m   +021.26.45.9  -30.32.39.6        209.5861       18.6784      -12.5958  5109139.532898  2006992.255752 -3239111.379568
    6    m021  m021      13.5 m   +021.26.26.9  -30.32.43.1       -297.0291      -89.4116      -14.5325  5109272.059019  2006500.019414 -3239203.485701
    7    m022  m022      13.5 m   +021.26.24.0  -30.32.32.5       -374.0562      238.3833      -16.0339  5109454.067853  2006488.741820 -3238920.413095
    8    m034  m034      13.5 m   +021.26.51.4  -30.32.33.5        356.7433      209.5211      -12.7966  5109175.834256  2007164.622574 -3238946.915796
    9    m041  m041      13.5 m   +021.26.27.2  -30.32.54.0       -288.6174     -423.8570      -13.6636  5109111.465262  2006445.988209 -3239491.955748
    10   m042  m042      13.5 m   +021.26.24.4  -30.32.47.4       -362.7833     -222.4880      -14.5362  5109233.138351  2006414.093264 -3239318.091862
    11   m050  m050      13.5 m   +021.25.20.9  -30.32.59.8      -2053.5171     -605.6775      -18.4460  5109666.453508  2004767.934259 -3239646.107249
    12   m051  m051      13.5 m   +021.26.06.0  -30.32.57.4       -851.3585     -531.4990      -16.2533  5109264.131845  2005901.393571 -3239583.340237
    13   m052  m052      13.5 m   +021.26.15.7  -30.33.09.7       -594.3019     -910.8179      -14.4035  5108992.203847  2006070.768233 -3239910.940391
    14   m060  m060      13.5 m   +021.28.46.5  -30.33.32.1       3419.1284    -1602.1789       -2.2768  5107206.755627  2009680.796911 -3240512.459326
    15   m061  m061      13.5 m   +021.26.37.4  -30.33.47.7        -17.4712    -2085.9876       -6.8131  5108231.343443  2006391.596905 -3240926.754178

    ```

    Notice that in this `listobs()` output, the first scans are the fields that will be used for calibration - they are observed before the target fields; what do you think: is there a reason to set up observations like that? Is it necessary? 
    With this information we can continue for the Pipeline. 

## Data Processing with ProcessMeerKAT

### Initial/Cross-Calibration with processMeerKAT

**The tutorial below is based on the MS `1525469431` with a few missing details, if your MS has all the details such as the DEEP2 data, you may jump some steps such as step 4 of the tutorial.** 

If you have, `ssh` into the ilifu cluster (slurm.ilifu.ac.za), and created a working directory somewhere on the filesystem (e.g. `/scratch/users/your_username/tutorial/` or `/scratch/users/your_username/tutorial/`).

1. Source `setup.sh`, which will add to your PATH and PYTHONPATH
    ```bash
    source /idia/software/pipelines/master/setup.sh
    ```
2. Build a config file, using verbose mode, and pointing to the Cluster Data Set
    ```bash
    processMeerKAT.py -B -C tutorial_config.txt -M /scratch3/users/walter/pipeline-training/1525469431/1525469431_sdp_l0_2025-09-03T09-10-19_s1f.ms -v
    ```
    If all is good, this should be your output, with different timestamps. and now you shou have the `tutorial_config.txt` available in your workspace.

    ```
    processMeerKAT.py -B -C tutorial_config.txt -M /scratch3/users/walter/pipeline-training/1525469431/1525469431_sdp_l0_2025-09-03T09-10-19_s1f.ms -v
    2025-09-17 10:52:11,534 INFO: Extracting field IDs from MeasurementSet "/scratch3/users/walter/pipeline-training/1525469431/1525469431_sdp_l0_2025-09-03T09-10-19_s1f.ms" using CASA.
    2025-09-17 10:52:11,534 DEBUG: Using the following command:
        srun --time=10 --mem=4GB --partition=Main --account=b03-idia-ag --qos qos-interactive singularity exec /idia/software/containers/casa-6.5.0-modular.sif  python /idia/software/pipelines/master/processMeerKAT/read_ms.py -B -M /scratch3/users/walter/pipeline-training/1525469431/1525469431_sdp_l0_2025-09-03T09-10-19_s1f.ms -C tutorial_config.txt -N 1 -t 8 -v
    srun: job 11686703 queued and waiting for resources
    srun: job 11686703 has been allocated resources
    2025-09-17 10:52:50	WARN	msmetadata_cmpt.cc::fieldsforintent	No intent 'CALIBRATE_FLUX' exists in this dataset.
    2025-09-17 10:52:49,998 ERROR: You must have a field with intent "CALIBRATE_FLUX". I only found ['CALIBRATE_AMPLI', 'CALIBRATE_PHASE', 'TARGET', 'UNKNOWN'] in dataset "/scratch3/users/walter/pipeline-training/1525469431/1525469431_sdp_l0_2025-09-03T09-10-19_s1f.ms".
    2025-09-17 10:52:49,998 INFO: [fields] section written to "tutorial_config.txt". Edit this section if you need to change field IDs (comma-seperated string for multiple IDs, not supported for calibrators).
    2025-09-17 10:52:50,024 WARNING: Only 2 polarisations present in '/scratch3/users/walter/pipeline-training/1525469431/1525469431_sdp_l0_2025-09-03T09-10-19_s1f.ms'. Any attempted polarisation calibration will fail, so setting dopol=False in [run] section of 'tutorial_config.txt'.
    2025-09-17 10:52:50,328 WARNING: Reference antenna 'm059' isn't present in input dataset '/scratch3/users/walter/pipeline-training/1525469431/1525469431_sdp_l0_2025-09-03T09-10-19_s1f.ms'. Antennas present are: ['m001', 'm002', 'm003', 'm006', 'm008', 'm015', 'm021', 'm022', 'm034', 'm041', 'm042', 'm050', 'm051', 'm052', 'm060', 'm061']. Try 'm052' or 'm005' if present, or ensure 'calcrefant=True' and 'calc_refant.py' script present in 'tutorial_config.txt'.
    2025-09-17 10:52:51,261 WARNING: The number of threads (1 node(s) x 8 task(s) = 8) is not ideal compared to the number of scans (32) for "/scratch3/users/walter/pipeline-training/1525469431/1525469431_sdp_l0_2025-09-03T09-10-19_s1f.ms".
    2025-09-17 10:52:51,262 WARNING: Config file has been updated to use 1 node(s) and 16 task(s) per node.
    2025-09-17 10:52:51,301 DEBUG: Overwritting [run] section in config file "tutorial_config.txt" with:
    {'dopol': False}.
    2025-09-17 10:52:51,315 DEBUG: Overwritting [slurm] section in config file "tutorial_config.txt" with:
    {'nodes': 1, 'ntasks_per_node': 16}.
    2025-09-17 10:52:51,323 DEBUG: Overwritting [fields] section in config file "tutorial_config.txt" with:
    {}.
    2025-09-17 10:52:51,331 DEBUG: Overwritting [crosscal] section in config file "tutorial_config.txt" with:
    {'spw': "'*:880.0~1680.0MHz'"}.
    2025-09-17 10:52:51,905 INFO: Config "tutorial_config.txt" generated.
    ```
    The purpose of this call is to read the input MS and extract information used to build the pipeline run, such as the field IDs corresponding to our different fields, and the number of scans (to check against the nodes and tasks per node, each of which is handled by a MPI worker - see step 3). See More details of this step [here](https://idia-pipelines.github.io/docs/processMeerKAT/deep-2-tutorial/).

3. Inspect the Created Config file which has the contents.

    ```
    [data]
    vis = '/scratch3/users/walter/pipeline-training/1525469431/1525469431_sdp_l0_2025-09-03T09-10-19_s1f.ms'

    [fields]
    bpassfield = ''
    fluxfield = ''
    phasecalfield = ''
    targetfields = ''
    extrafields = ''

    [slurm]
    nodes = 1
    ntasks_per_node = 16
    plane = 1
    mem = 232
    partition = 'Main'
    exclude = ''
    time = '12:00:00'
    submit = False
    container = '/idia/software/containers/casa-6.5.0-modular.sif'
    mpi_wrapper = 'mpirun'
    name = ''
    dependencies = ''
    account = 'b03-idia-ag'
    reservation = ''
    modules = ['openmpi/4.0.3']
    verbose = True
    precal_scripts = [('calc_refant.py', False, ''), ('partition.py', True, '')]
    postcal_scripts = [('concat.py', False, ''), ('plotcal_spw.py', False, '')]
    scripts = [('validate_input.py', False, ''), ('flag_round_1.py', True, ''), ('calc_refant.py', False, ''), ('setjy.py', True, ''), ('xx_yy_solve.py', False, ''), ('xx_yy_apply.py', True, ''), ('flag_round_2.py', True, ''), ('xx_yy_solve.py', False, ''), ('xx_yy_apply.py', True, ''), ('split.py', True, ''), ('quick_tclean.py', True, '')]

    [crosscal]
    minbaselines = 4                  # Minimum number of baselines to use while calibrating
    chanbin = 1                       # Number of channels to average before calibration (during partition)
    width = 1                         # Number of channels to (further) average after calibration (during split)
    timeavg = '8s'                    # Time interval to average after calibration (during split)
    createmms = True                  # Create MMS (True) or MS (False) for cross-calibration during partition
    keepmms = True                    # Output MMS (True) or MS (False) during split
    spw = '*:880.0~1680.0MHz'
    nspw = 11                         # Number of spectral windows to split into
    calcrefant = False                # Calculate reference antenna in program (overwrites 'refant')
    refant = 'm059'                   # Reference antenna name / number
    standard = 'Stevens-Reynolds 2016'# Flux density standard for setjy
    badants = []                      # List of bad antenna numbers (to flag)
    badfreqranges = [ '933~960MHz',   # List of bad frequency ranges (to flag)
        '1163~1299MHz',
        '1524~1630MHz']

    [run]
    continue = True
    dopol = False
    ```

    This config file is organized into five sections: `data`, `fields`, `slurm`, `crosscal`, and `run`. The field IDs listed in the `fields` section should be automatically extracted by the pipeline and assigned as follows:
    - Field 0 for the bandpass calibrator
    - Field 0 for the total flux calibrator
    - Field 2 for the phase calibrator
    - Field 3 for the science target (e.g., the DEEP2 field)
    - Field 1 for an extra calibrator, used for applying solutions and generating a quick-look image
    
    Only the target and extra fields can include multiple field IDs, separated by commas. If a field matching the required intent is not found, the pipeline will display a warning and select the total flux calibrator field by default. If the total flux calibrator is missing, the process will terminate with an error. For other section explanation please see visit [DEEP2 tutorial](https://idia-pipelines.github.io/docs/processMeerKAT/deep-2-tutorial/)

    **Note**: The [Fields] section of our config are empty strings and this should be automatically populated by the pipeline. If you check/build DEEP2 config file, you’ll notice that fields are automatically populated. The issue here is seen in step 2 above, the output includes an error:
    `ERROR: You must have a field with intent "CALIBRATE_FLUX". I only found ['CALIBRATE_AMPLI', 'CALIBRATE_PHASE', 'TARGET', 'UNKNOWN']`
    which does not occur with the DEEP2 data. **This highlights the importance of understanding your dataset before launching the pipeline**. While the default configuration may work in many cases, there are situations where you will need to inspect and manually update the config file to ensure proper calibration and processing

4. Edit the config file 

    Here we update the config file to add the FIELDS and also update the reference antenna. As you can see from `listobs()` our data does not include the m059 antenna and the config file has specified this as the reference antenna. For this tutorial I am selectiong `m052` as my reference antenna, why...?

    Our updated config should now be like
        ```
        [data]
        vis = '/scratch3/users/walter/pipeline-training/1525469431/1525469431_sdp_l0_2025-09-03T09-10-19_s1f.ms'

        [fields]
        bpassfield = 'J1939-6342'
        fluxfield = 'J1939-6342'
        phasecalfield = 'J1939-6342'
        targetfields = 'ACT-CLJ2023.3-5535'
        extrafields = ''

        [slurm]
        nodes = 1
        ntasks_per_node = 16
        plane = 1
        mem = 232
        partition = 'Main'
        exclude = ''
        time = '12:00:00'
        submit = False
        container = '/idia/software/containers/casa-6.5.0-modular.sif'
        mpi_wrapper = 'mpirun'
        name = ''
        dependencies = ''
        account = 'b03-idia-ag'
        reservation = ''
        modules = ['openmpi/4.0.3']
        verbose = True
        precal_scripts = [('calc_refant.py', False, ''), ('partition.py', True, '')]
        postcal_scripts = [('concat.py', False, ''), ('plotcal_spw.py', False, '')]
        scripts = [('validate_input.py', False, ''), ('flag_round_1.py', True, ''), ('calc_refant.py', False, ''), ('setjy.py', True, ''), ('xx_yy_solve.py', False, ''), ('xx_yy_apply.py', True, ''), ('flag_round_2.py', True, ''), ('xx_yy_solve.py', False, ''), ('xx_yy_apply.py', True, ''), ('split.py', True, ''), ('quick_tclean.py', True, '')]

        [crosscal]
        minbaselines = 4                  # Minimum number of baselines to use while calibrating
        chanbin = 1                       # Number of channels to average before calibration (during partition)
        width = 1                         # Number of channels to (further) average after calibration (during split)
        timeavg = '8s'                    # Time interval to average after calibration (during split)
        createmms = True                  # Create MMS (True) or MS (False) for cross-calibration during partition
        keepmms = True                    # Output MMS (True) or MS (False) during split
        spw = '*:880.0~1680.0MHz'
        nspw = 11                         # Number of spectral windows to split into
        calcrefant = False                # Calculate reference antenna in program (overwrites 'refant')
        refant = 'm052'                   # Reference antenna name / number
        standard = 'Stevens-Reynolds 2016'# Flux density standard for setjy
        badants = []                      # List of bad antenna numbers (to flag)
        badfreqranges = [ '933~960MHz',   # List of bad frequency ranges (to flag)
            '1163~1299MHz',
            '1524~1630MHz']

        [run]
        continue = True
        dopol = False
        ```
5. Running the pipeline using the config file above

    ```bash
    processMeerKAT.py -R -C tutorial_config.txt
    ```
    This should produce an output like
    ```
    2025-09-17 11:48:04,902 INFO: Won't process spw '*:1170.909090909091~1243.6363636363635MHz', since it's completely encompassed by bad frequency range 'MHz'.
    2025-09-17 11:48:04,902 INFO: Won't process spw '*:1534.5454545454545~1607.2727272727273MHz', since it's completely encompassed by bad frequency range 'MHz'.
    2025-09-17 11:48:04,910 INFO: Making 9 directories for SPWs (['*:880.0~952.7272727272727MHz', '*:952.7272727272727~1025.4545454545455MHz', '*:1025.4545454545455~1098.1818181818182MHz', '*:1098.1818181818182~1170.909090909091MHz', '*:1243.6363636363635~1316.3636363636365MHz', '*:1316.3636363636365~1389.090909090909MHz', '*:1389.090909090909~1461.818181818182MHz', '*:1461.8181818181818~1534.5454545454545MHz', '*:1607.2727272727273~1680.0MHz']) and copying 'tutorial_config.txt' to each of them.
    2025-09-17 11:48:05,809 DEBUG: Copying 'tutorial_config.txt' to '.config.tmp', and using this to run pipeline.
    2025-09-17 11:48:05,811 WARNING: Changing [slurm] section in your config will have no effect unless you [-R --run] again.
    2025-09-17 11:48:05,822 DEBUG: Wrote sbatch file "partition.sbatch"
    2025-09-17 11:48:05,824 DEBUG: Wrote sbatch file "concat.sbatch"
    2025-09-17 11:48:05,828 DEBUG: Wrote sbatch file "plotcal_spw.sbatch"
    2025-09-17 11:48:05,978 DEBUG: Copying './tutorial_config.txt' to '.config.tmp', and using this to run pipeline.
    2025-09-17 11:48:05,981 DEBUG: Wrote sbatch file "validate_input.sbatch"
    2025-09-17 11:48:05,982 DEBUG: Wrote sbatch file "flag_round_1.sbatch"
    2025-09-17 11:48:05,983 DEBUG: Wrote sbatch file "setjy.sbatch"
    2025-09-17 11:48:05,984 DEBUG: Wrote sbatch file "xx_yy_solve.sbatch"
    2025-09-17 11:48:05,985 DEBUG: Wrote sbatch file "xx_yy_apply.sbatch"
    2025-09-17 11:48:05,985 DEBUG: Wrote sbatch file "flag_round_2.sbatch"
    2025-09-17 11:48:05,997 DEBUG: Wrote sbatch file "xx_yy_solve.sbatch"
    2025-09-17 11:48:06,007 DEBUG: Wrote sbatch file "xx_yy_apply.sbatch"
    2025-09-17 11:48:06,008 DEBUG: Wrote sbatch file "split.sbatch"
    2025-09-17 11:48:06,008 DEBUG: Wrote sbatch file "quick_tclean.sbatch"
    2025-09-17 11:48:06,018 INFO: Master script "submit_pipeline.sh" written in "880.0~952.7272727272727MHz", but will not run.
    2025-09-17 11:48:06,134 DEBUG: Copying './tutorial_config.txt' to '.config.tmp', and using this to run pipeline.
    2025-09-17 11:48:06,137 DEBUG: Wrote sbatch file "validate_input.sbatch"
    2025-09-17 11:48:06,137 DEBUG: Wrote sbatch file "flag_round_1.sbatch"
    2025-09-17 11:48:06,138 DEBUG: Wrote sbatch file "setjy.sbatch"
    2025-09-17 11:48:06,139 DEBUG: Wrote sbatch file "xx_yy_solve.sbatch"
    2025-09-17 11:48:06,139 DEBUG: Wrote sbatch file "xx_yy_apply.sbatch"
    2025-09-17 11:48:06,140 DEBUG: Wrote sbatch file "flag_round_2.sbatch"
    2025-09-17 11:48:06,148 DEBUG: Wrote sbatch file "xx_yy_solve.sbatch"
    2025-09-17 11:48:06,156 DEBUG: Wrote sbatch file "xx_yy_apply.sbatch"
    2025-09-17 11:48:06,161 DEBUG: Wrote sbatch file "split.sbatch"
    2025-09-17 11:48:06,162 DEBUG: Wrote sbatch file "quick_tclean.sbatch"
    .
    .
    .
    ```
    A number of sbatch files have now been written to your `working` directory, each of which corresponds to the python script in the list of scripts set by the scripts parameter in our config file. Our config file was copied to .`config.tmp`, which is the config file written and edited by the pipeline, which **the user should not touch**. A `logs` directory was created, which will store the **CASA and SLURM** log files. Lastly, a bash script called `submit_pipeline.sh` was written, however, this script was not run, since we set `submit = False` in our config file (to immediately submit to the SLURM queue, you can change this in your config file, or by using option [-s --submit] when you build your config file with processMeerKAT.py). Normally, we would run `./submit_pipeline.sh` to run the pipeline, and return later when it is completed. However, we will look at later, as we first want to get a handle of how the pipeline works.

    ```bash
    walter@slurm-login:/scratch3/users/walter/pipeline-training/tutorial-1525469431$ ls -l
    1025.4545454545455~1098.1818181818182MHz  
    1389.090909090909~1461.818181818182MHz    
    952.7272727272727~1025.4545454545455MHz  
    1098.1818181818182~1170.909090909091MHz   
    1461.8181818181818~1534.5454545454545MHz  
    1243.6363636363635~1316.3636363636365MHz  
    1607.2727272727273~1680.0MHz 
    1316.3636363636365~1389.090909090909MHz   
    880.0~952.7272727272727MHz 
    partition.sbatch
    casa-20250917-105225.log                 
    plotcal_spw.sbatch             
    concat.sbatch                            
    submit_pipeline.sh               
    logs                                     
    tutorial_config.txt
    ```

6. Submitting the Job
    The step above will create `submit_pipeline.sh`, which you can then run with `./submit_pipeline.sh` to submit all pipeline jobs to the SLURM queue.
    Once the jobs have been submitted, you can check their status in the SLURM queue by running: `squeu -u walter` where `walter` is my ilifu username
    ```
    JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
          11679108   Jupyter jupyter-   walter  R   21:05:17      1 jupyter-013
          11686880      Main quick_tc   walter PD       0:00      1 (Dependency)
          11686870      Main quick_tc   walter PD       0:00      1 (Dependency)
          11686860      Main quick_tc   walter PD       0:00      1 (Dependency)
          11686850      Main quick_tc   walter PD       0:00      1 (Dependency)
          11686840      Main quick_tc   walter PD       0:00      1 (Dependency)
          11686830      Main quick_tc   walter PD       0:00      1 (Dependency)
          11686820      Main quick_tc   walter PD       0:00      1 (Dependency)
          11686810      Main quick_tc   walter PD       0:00      1 (Dependency)
          11686800      Main quick_tc   walter PD       0:00      1 (Dependency)
        11686782_8      Main partitio   walter PD       0:00      1 (Priority)
        11686782_7      Main partitio   walter PD       0:00      1 (Priority)
        11686782_6      Main partitio   walter PD       0:00      1 (Priority)
        11686782_5      Main partitio   walter PD       0:00      1 (Priority)
        11686782_4      Main partitio   walter PD       0:00      1 (Priority)
        11686782_3      Main partitio   walter PD       0:00      1 (Priority)
        11686782_2      Main partitio   walter PD       0:00      1 (Priority)
        11686782_1      Main partitio   walter PD       0:00      1 (Priority)
        11686782_0      Main partitio   walter PD       0:00      1 (Priority)
          11686879      Main    split   walter PD       0:00      1 (Dependency)
          11686878      Main xx_yy_ap   walter PD       0:00      1 (Dependency)
          11686876      Main flag_rou   walter PD       0:00      1 (Dependency)
          11686875      Main xx_yy_ap   walter PD       0:00      1 (Dependency)
    ```
    Other convenience scripts are also created that allow you to monitor and (if necessary) kill the jobs.

    - `summary.sh` provides a brief overview of the status of the jobs in the pipeline
    - `findErrors.sh` checks the log files for commonly reported errors (after the jobs have run)
    - `killJobs.sh` kills all the jobs from the current run of the pipeline, ignoring any other (unrelated) jobs you might have running.
    - `cleanup.sh` wipes all the intermediate data products created by the pipeline. This is intended to be launched after the pipeline has run and the output is verified to be good.

7. Job Monitoring 
    Using the `./summary.sh` script.

    ```
    walter@compute-001:/scratch3/users/walter/pipeline-training/tutorial-1525469431$ ./summary.sh
    SPW #1: /scratch3/users/walter/pipeline-training/tutorial-1525469431/880.0~952.7272727272727MHz
    JobID           JobName          Partition    Elapsed NNodes NTasks NCPUS  MaxDiskRead MaxDiskWrite             NodeList   TotalCPU    CPUTime     MaxRSS      State ExitCode
    --------------- --------------- ---------- ---------- ------ ------ ----- ------------ ------------ -------------------- ---------- ---------- ---------- ---------- --------
    -----------------------------------------------------------------------------------------------------------------------------------------------------------------------------
    SPW #2: /scratch3/users/walter/pipeline-training/tutorial-1525469431/952.7272727272727~1025.4545454545455MHz
    JobID           JobName          Partition    Elapsed NNodes NTasks NCPUS  MaxDiskRead MaxDiskWrite             NodeList   TotalCPU    CPUTime     MaxRSS      State ExitCode
    --------------- --------------- ---------- ---------- ------ ------ ----- ------------ ------------ -------------------- ---------- ---------- ---------- ---------- --------
    -----------------------------------------------------------------------------------------------------------------------------------------------------------------------------
    SPW #3: /scratch3/users/walter/pipeline-training/tutorial-1525469431/1025.4545454545455~1098.1818181818182MHz
    JobID           JobName          Partition    Elapsed NNodes NTasks NCPUS  MaxDiskRead MaxDiskWrite             NodeList   TotalCPU    CPUTime     MaxRSS      State ExitCode
    --------------- --------------- ---------- ---------- ------ ------ ----- ------------ ------------ -------------------- ---------- ---------- ---------- ---------- --------
    -----------------------------------------------------------------------------------------------------------------------------------------------------------------------------
    SPW #4: /scratch3/users/walter/pipeline-training/tutorial-1525469431/1098.1818181818182~1170.909090909091MHz
    JobID           JobName          Partition    Elapsed NNodes NTasks NCPUS  MaxDiskRead MaxDiskWrite             NodeList   TotalCPU    CPUTime     MaxRSS      State ExitCode
    --------------- --------------- ---------- ---------- ------ ------ ----- ------------ ------------ -------------------- ---------- ---------- ---------- ---------- --------
    -----------------------------------------------------------------------------------------------------------------------------------------------------------------------------
    SPW #5: /scratch3/users/walter/pipeline-training/tutorial-1525469431/1243.6363636363635~1316.3636363636365MHz
    JobID           JobName          Partition    Elapsed NNodes NTasks NCPUS  MaxDiskRead MaxDiskWrite             NodeList   TotalCPU    CPUTime     MaxRSS      State ExitCode
    --------------- --------------- ---------- ---------- ------ ------ ----- ------------ ------------ -------------------- ---------- ---------- ---------- ---------- --------
    -----------------------------------------------------------------------------------------------------------------------------------------------------------------------------
    SPW #6: /scratch3/users/walter/pipeline-training/tutorial-1525469431/1316.3636363636365~1389.090909090909MHz
    JobID           JobName          Partition    Elapsed NNodes NTasks NCPUS  MaxDiskRead MaxDiskWrite             NodeList   TotalCPU    CPUTime     MaxRSS      State ExitCode
    --------------- --------------- ---------- ---------- ------ ------ ----- ------------ ------------ -------------------- ---------- ---------- ---------- ---------- --------
    -----------------------------------------------------------------------------------------------------------------------------------------------------------------------------
    SPW #7: /scratch3/users/walter/pipeline-training/tutorial-1525469431/1389.090909090909~1461.818181818182MHz
    JobID           JobName          Partition    Elapsed NNodes NTasks NCPUS  MaxDiskRead MaxDiskWrite             NodeList   TotalCPU    CPUTime     MaxRSS      State ExitCode
    --------------- --------------- ---------- ---------- ------ ------ ----- ------------ ------------ -------------------- ---------- ---------- ---------- ---------- --------
    -----------------------------------------------------------------------------------------------------------------------------------------------------------------------------
    SPW #8: /scratch3/users/walter/pipeline-training/tutorial-1525469431/1461.8181818181818~1534.5454545454545MHz
    JobID           JobName          Partition    Elapsed NNodes NTasks NCPUS  MaxDiskRead MaxDiskWrite             NodeList   TotalCPU    CPUTime     MaxRSS      State ExitCode
    --------------- --------------- ---------- ---------- ------ ------ ----- ------------ ------------ -------------------- ---------- ---------- ---------- ---------- --------
    -----------------------------------------------------------------------------------------------------------------------------------------------------------------------------
    SPW #9: /scratch3/users/walter/pipeline-training/tutorial-1525469431/1607.2727272727273~1680.0MHz
    JobID           JobName          Partition    Elapsed NNodes NTasks NCPUS  MaxDiskRead MaxDiskWrite             NodeList   TotalCPU    CPUTime     MaxRSS      State ExitCode
    --------------- --------------- ---------- ---------- ------ ------ ----- ------------ ------------ -------------------- ---------- ---------- ---------- ---------- --------
    -----------------------------------------------------------------------------------------------------------------------------------------------------------------------------
    All SPWs: /scratch3/users/walter/pipeline-training/tutorial-1525469431
    JobID           JobName          Partition    Elapsed NNodes NTasks NCPUS  MaxDiskRead MaxDiskWrite             NodeList   TotalCPU    CPUTime     MaxRSS      State ExitCode
    --------------- --------------- ---------- ---------- ------ ------ ----- ------------ ------------ -------------------- ---------- ---------- ---------- ---------- --------
    11686782_0      partition             Main   00:02:35      1           32                                    compute-212  04:00.061   01:22:40             COMPLETED      0:0
    11686782_0.bat+ batch                        00:02:35      1      1    32       82.29G        6.75G          compute-212  04:00.058   01:22:40     79.77G  COMPLETED      0:0
    11686782_0.ext+ extern                       00:02:35      1      1    32        0.01M        0.00M          compute-212  00:00.003   01:22:40      0.00G  COMPLETED      0:0
    11686782_1      partition             Main   00:01:30      1           32                                    compute-212  03:17.231   00:48:00             COMPLETED      0:0
    11686782_1.bat+ batch                        00:01:30      1      1    32       82.29G        6.75G          compute-212  03:17.228   00:48:00     33.25G  COMPLETED      0:0
    11686782_1.ext+ extern                       00:01:30      1      1    32        0.01M        0.00M          compute-212  00:00.003   00:48:00      0.00G  COMPLETED      0:0
    11686782_2      partition             Main   00:01:10      1           32                                    compute-212  02:40.337   00:37:20             COMPLETED      0:0
    11686782_2.bat+ batch                        00:01:10      1      1    32       82.29G        6.75G          compute-212  02:40.334   00:37:20     12.67G  COMPLETED      0:0
    11686782_2.ext+ extern                       00:01:10      1      1    32        0.01M        0.00M          compute-212  00:00.003   00:37:20      0.00G  COMPLETED      0:0
    11686782_3      partition             Main   00:01:40      1           32                                    compute-212  03:32.799   00:53:20             COMPLETED      0:0
    11686782_3.bat+ batch                        00:01:40      1      1    32       82.28G        6.75G          compute-212  03:32.796   00:53:20     14.25G  COMPLETED      0:0
    11686782_3.ext+ extern                       00:01:40      1      1    32        0.01M        0.00M          compute-212  00:00.003   00:53:20      0.00G  COMPLETED      0:0
    11686782_4      partition             Main   00:01:09      1           32                                    compute-212  02:34.888   00:36:48             COMPLETED      0:0
    11686782_4.bat+ batch                        00:01:09      1      1    32       82.28G        6.75G          compute-212  02:34.885   00:36:48     12.78G  COMPLETED      0:0
    11686782_4.ext+ extern                       00:01:09      1      1    32        0.01M        0.00M          compute-212  00:00.003   00:36:48      0.00G  COMPLETED      0:0
    11686782_5      partition             Main   00:01:15      1           32                                    compute-212  02:40.802   00:40:00             COMPLETED      0:0
    11686782_5.bat+ batch                        00:01:15      1      1    32       82.29G        6.75G          compute-212  02:40.798   00:40:00     12.81G  COMPLETED      0:0
    11686782_5.ext+ extern                       00:01:15      1      1    32        0.01M        0.00M          compute-212  00:00.003   00:40:00      0.00G  COMPLETED      0:0
    11686782_6      partition             Main   00:01:16      1           32                                    compute-212  02:40.858   00:40:32             COMPLETED      0:0
    11686782_6.bat+ batch                        00:01:16      1      1    32       82.29G        6.75G          compute-212  02:40.855   00:40:32     12.78G  COMPLETED      0:0
    11686782_6.ext+ extern                       00:01:16      1      1    32        0.01M        0.00M          compute-212  00:00.003   00:40:32      0.00G  COMPLETED      0:0
    11686782_7      partition             Main   00:01:58      1           32                                    compute-008  03:09.978   01:02:56             COMPLETED      0:0
    11686782_7.bat+ batch                        00:01:58      1      1    32       82.29G        6.75G          compute-008  03:09.977   01:02:56     90.84G  COMPLETED      0:0
    11686782_7.ext+ extern                       00:01:58      1      1    32        0.01M        0.00M          compute-008  00:00.001   01:02:56      0.00G  COMPLETED      0:0
    11686782_8      partition             Main   00:01:01      1           32                                    compute-212  02:19.556   00:32:32             COMPLETED      0:0
    11686782_8.bat+ batch                        00:01:01      1      1    32       82.28G        6.75G          compute-212  02:19.552   00:32:32     12.69G  COMPLETED      0:0
    11686782_8.ext+ extern                       00:01:01      1      1    32        0.01M        0.00M          compute-212  00:00.003   00:32:32      0.00G  COMPLETED      0:0
    11686881        concat                Main   00:05:37      1            1                                    compute-103   00:00:00   00:05:37               RUNNING      0:0
    11686881.batch  batch                        00:05:37      1      1     1                                    compute-103   00:00:00   00:05:37               RUNNING      0:0
    11686881.extern extern                       00:05:37      1      1     1                                    compute-103   00:00:00   00:05:37               RUNNING      0:0
    11686881.0      singularity                  00:05:37      1      1     1                                    compute-103   00:00:00   00:05:37               RUNNING      0:0
    11686882        plotcal_spw           Main   00:00:00      1            0                                  None assigned   00:00:00   00:00:00               PENDING      0:0
    ```

    You can also view a summary of pipeline jobs for a specific spectral window (SPW). This allows you to monitor progress and review results for each SPW individually. For example, to check the job summary for SPW #9, navigate to its directory; `/scratch3/users/walter/pipeline-training/tutorial-1525469431/1607.2727272727273~1680.0MHz`.

    ```
    cd /scratch3/users/walter/pipeline-training/tutorial-1525469431/1607.2727272727273~1680.0MHz

    ./summary.sh
    ```
    You can also select any other SPW directory from the summary above and run the same command.
    The output will provide a detailed summary of the pipeline jobs for that SPW.

    ```
    walter@compute-001:/scratch3/users/walter/pipeline-training/tutorial-1525469431/1607.2727272727273~1680.0MHz$ ./summary.sh
    JobID           JobName          Partition    Elapsed NNodes NTasks NCPUS  MaxDiskRead MaxDiskWrite             NodeList   TotalCPU    CPUTime     MaxRSS      State ExitCode
    --------------- --------------- ---------- ---------- ------ ------ ----- ------------ ------------ -------------------- ---------- ---------- ---------- ---------- --------
    11686871        validate_input        Main   00:00:32      1            1                                    compute-201  00:06.339   00:00:32             COMPLETED      0:0
    11686871.batch  batch                        00:00:32      1      1     1        0.99M        0.00M          compute-201  00:00.096   00:00:32      0.00G  COMPLETED      0:0
    11686871.extern extern                       00:00:32      1      1     1        0.01M        0.00M          compute-201  00:00.003   00:00:32      0.00G  COMPLETED      0:0
    11686871.0      singularity                  00:00:31      1      1     1        0.02G        0.01M          compute-201  00:06.239   00:00:31      2.20G  COMPLETED      0:0
    11686872        flag_round_1          Main   00:01:40      1           16                                    compute-008  05:34.903   00:26:40             COMPLETED      0:0
    11686872.batch  batch                        00:01:40      1      1    16        7.98G        0.53G          compute-008  05:34.901   00:26:40     15.81G  COMPLETED      0:0
    11686872.extern extern                       00:01:40      1      1    16        0.01M        0.00M          compute-008  00:00.001   00:26:40      0.00G  COMPLETED      0:0
    11686873        setjy                 Main   00:01:59      1           16                                    compute-256  01:23.211   00:31:44             COMPLETED      0:0
    11686873.batch  batch                        00:01:59      1      1    16        7.01G        6.57G          compute-256  01:23.207   00:31:44     15.39G  COMPLETED      0:0
    11686873.extern extern                       00:01:59      1      1    16        0.01M        0.00M          compute-256  00:00.003   00:31:44      0.00G  COMPLETED      0:0
    11686874        xx_yy_solve           Main   00:00:56      1            1                                    compute-201  00:33.945   00:00:56             COMPLETED      0:0
    11686874.batch  batch                        00:00:56      1      1     1        0.99M        0.00M          compute-201  00:00.095   00:00:56      0.00G  COMPLETED      0:0
    11686874.extern extern                       00:00:56      1      1     1        0.01M        0.00M          compute-201  00:00.003   00:00:56          0  COMPLETED      0:0
    11686874.0      singularity                  00:00:55      1      1     1        0.88G        0.19M          compute-201  00:33.846   00:00:55      3.84G  COMPLETED      0:0
    11686875        xx_yy_apply           Main   00:01:19      1           16                                    compute-103  02:28.943   00:21:04             COMPLETED      0:0
    11686875.batch  batch                        00:01:19      1      1    16        5.27G        3.94G          compute-103  02:28.940   00:21:04     17.36G  COMPLETED      0:0
    11686875.extern extern                       00:01:19      1      1    16        0.01M        0.00M          compute-103  00:00.003   00:21:04      0.00G  COMPLETED      0:0
    11686876        flag_round_2          Main   00:01:59      1           16                                    compute-017  09:01.903   00:31:44             COMPLETED      0:0
    11686876.batch  batch                        00:01:59      1      1    16        7.70G        0.38G          compute-017  09:01.901   00:31:44     14.58G  COMPLETED      0:0
    11686876.extern extern                       00:01:59      1      1    16        0.01M        0.00M          compute-017  00:00.001   00:31:44      0.00G  COMPLETED      0:0
    11686877        xx_yy_solve           Main   00:01:05      1            1                                    compute-202  00:40.839   00:01:05             COMPLETED      0:0
    11686877.batch  batch                        00:01:05      1      1     1        0.00G        0.02M          compute-202  00:00.112   00:01:05      0.00G  COMPLETED      0:0
    11686877.extern extern                       00:01:05      1      1     1        0.01M        0.00M          compute-202  00:00.003   00:01:05      0.00G  COMPLETED      0:0
    11686877.0      singularity                  00:01:05      1      1     1        1.57G        0.69M          compute-202  00:40.723   00:01:05      4.01G  COMPLETED      0:0
    11686878        xx_yy_apply           Main   00:00:42      1           16                                    compute-017  02:30.305   00:11:12             COMPLETED      0:0
    11686878.batch  batch                        00:00:42      1      1    16        9.60G        3.75G          compute-017  02:30.304   00:11:12     15.29G  COMPLETED      0:0
    11686878.extern extern                       00:00:42      1      1    16        0.01M        0.00M          compute-017  00:00.001   00:11:12      0.00G  COMPLETED      0:0
    11686879        split                 Main   00:00:54      1           16                                    compute-017  01:54.254   00:14:24             COMPLETED      0:0
    11686879.batch  batch                        00:00:54      1      1    16        5.56G        3.46G          compute-017  01:54.252   00:14:24     11.61G  COMPLETED      0:0
    11686879.extern extern                       00:00:54      1      1    16        0.01M        0.00M          compute-017  00:00.001   00:14:24      0.00G  COMPLETED      0:0
    11686880        quick_tclean          Main   00:09:46      1           32                                    compute-230  10:37.965   05:12:32             COMPLETED      0:0
    11686880.batch  batch                        00:09:46      1      1    32       44.77G       17.15G          compute-230  10:37.961   05:12:32     18.40G  COMPLETED      0:0
    11686880.extern extern                       00:09:46      1      1    32        0.01M        0.00M          compute-230  00:00.003   05:12:32      0.00G  COMPLETED      0:0
    ```

    As shown above, all jobs for this SPW have completed successfully. If any job had failed, it would be clearly indicated in the summary output, allowing you to review the logs and investigate the cause of the failure.

**This concludes the First Part of the Tutorial** 


8. If we updated only **FIELDS** and the **reference antenna** was not updated to m052—the available antenna—in the configuration file, running the pipeline without these changes would result in a failure or errors related to the missing reference antenna. **This underscores the importance of verifying and correctly setting the reference antenna in the configuration before executing the pipeline.**
Below is an example case where the correct reference antenna was missing.
The full summary (`./summary.sh`) of that failed processing is shown below.
    ```
    walter@compute-001:~/scratch3/users/walter/pipeline-training/1525469431$ ./summary.sh
    SPW #1: /users/walter/scratch3/users/walter/pipeline-training/1525469431/880.0~952.7272727272727MHz
    JobID           JobName          Partition    Elapsed NNodes NTasks NCPUS  MaxDiskRead MaxDiskWrite             NodeList   TotalCPU    CPUTime     MaxRSS      State ExitCode
    --------------- --------------- ---------- ---------- ------ ------ ----- ------------ ------------ -------------------- ---------- ---------- ---------- ---------- --------
    11534134        validate_input        Main   00:00:41      1            1                                    compute-203  00:07.258   00:00:41                FAILED      1:0
    11534134.batch  batch                        00:00:41      1      1     1        0.99M        0.00M          compute-203  00:00.085   00:00:41      0.01G     FAILED      1:0
    11534134.0      singularity                  00:00:41      1      1     1        0.04G        0.01M          compute-203  00:07.170   00:00:41      3.86G     FAILED      1:0
    11534135        flag_round_1          Main   00:00:00      1            0                                  None assigned   00:00:00   00:00:00             CANCELLED      0:0
    11534136        setjy                 Main   00:00:00      1            0                                  None assigned   00:00:00   00:00:00             CANCELLED      0:0
    11534137        xx_yy_solve           Main   00:00:00      1            0                                  None assigned   00:00:00   00:00:00             CANCELLED      0:0
    11534138        xx_yy_apply           Main   00:00:00      1            0                                  None assigned   00:00:00   00:00:00             CANCELLED      0:0
    11534139        flag_round_2          Main   00:00:00      1            0                                  None assigned   00:00:00   00:00:00             CANCELLED      0:0
    11534140        xx_yy_solve           Main   00:00:00      1            0                                  None assigned   00:00:00   00:00:00             CANCELLED      0:0
    11534141        xx_yy_apply           Main   00:00:00      1            0                                  None assigned   00:00:00   00:00:00             CANCELLED      0:0
    11534142        split                 Main   00:00:00      1            0                                  None assigned   00:00:00   00:00:00             CANCELLED      0:0
    11534143        quick_tclean          Main   00:00:00      1            0                                  None assigned   00:00:00   00:00:00             CANCELLED      0:0
    -----------------------------------------------------------------------------------------------------------------------------------------------------------------------------
    SPW #2: /users/walter/scratch3/users/walter/pipeline-training/1525469431/952.7272727272727~1025.4545454545455MHz
    JobID           JobName          Partition    Elapsed NNodes NTasks NCPUS  MaxDiskRead MaxDiskWrite             NodeList   TotalCPU    CPUTime     MaxRSS      State ExitCode
    --------------- --------------- ---------- ---------- ------ ------ ----- ------------ ------------ -------------------- ---------- ---------- ---------- ---------- --------
    11534144        validate_input        Main   00:00:27      1            1                                    compute-203  00:05.019   00:00:27                FAILED      1:0
    11534144.batch  batch                        00:00:27      1      1     1        0.99M        0.00M          compute-203  00:00.123   00:00:27      0.10G     FAILED      1:0
    11534144.0      singularity                  00:00:26      1      1     1        0.03G        0.01M          compute-203  00:04.892   00:00:26      2.21G     FAILED      1:0
    11534145        flag_round_1          Main   00:00:00      1            0                                  None assigned   00:00:00   00:00:00             CANCELLED      0:0
    11534146        setjy                 Main   00:00:00      1            0                                  None assigned   00:00:00   00:00:00             CANCELLED      0:0
    11534147        xx_yy_solve           Main   00:00:00      1            0                                  None assigned   00:00:00   00:00:00             CANCELLED      0:0
    11534148        xx_yy_apply           Main   00:00:00      1            0                                  None assigned   00:00:00   00:00:00             CANCELLED      0:0
    11534149        flag_round_2          Main   00:00:00      1            0                                  None assigned   00:00:00   00:00:00             CANCELLED      0:0
    11534150        xx_yy_solve           Main   00:00:00      1            0                                  None assigned   00:00:00   00:00:00             CANCELLED      0:0
    11534151        xx_yy_apply           Main   00:00:00      1            0                                  None assigned   00:00:00   00:00:00             CANCELLED      0:0
    11534152        split                 Main   00:00:00      1            0                                  None assigned   00:00:00   00:00:00             CANCELLED      0:0
    11534153        quick_tclean          Main   00:00:00      1            0                                  None assigned   00:00:00   00:00:00             CANCELLED      0:0
    -----------------------------------------------------------------------------------------------------------------------------------------------------------------------------

    ```
    If we select the first SPW and inspect the logs, we can navigate to its directory using: 
    ```
    cd /users/walter/scratch3/users/walter/pipeline-training/1525469431/880.0~952.7272727272727MHz
    ```
    Inside this directory, the log files are located in the `logs` folder. From the summary, it is evident that the `validate_input` step failed, so we will examine the corresponding log file:
    ```
    cat logs/validate_input-11534154.err
    ```
    Where the output would look like 
    ```
    2025-09-03 13:05:48,645 INFO: This is version 2.0 of the pipeline - commit ID f4c7c719e65cf07a2d4609dededd825cb8c8f202

    2025-09-03 13:06:10,214 ERROR: Exception found in the pipeline of type <class 'ValueError'>: Reference antenna 'm059' isn't present in input dataset '1525469431_sdp_l0_2025-09-03T09-10-19_s1f.1025.4545454545455~1098.1818181818182MHz.mms'. Antennas present are: ['m001', 'm002', 'm003', 'm006', 'm008', 'm015', 'm021', 'm022', 'm034', 'm041', 'm042', 'm050', 'm051', 'm052', 'm060', 'm061']. Try 'm052' or 'm005' if present, or ensure 'calcrefant=True' and 'calc_refant.py' script present in '.config.tmp'.
    2025-09-03 13:06:10,215 ERROR: Traceback (most recent call last):
    File "/idia/software/pipelines/master/processMeerKAT/bookkeeping.py", line 341, in run_script
        func(args,taskvals)
    File "/idia/software/pipelines/master/processMeerKAT/validate_input.py", line 50, in main
        read_ms.check_refant(MS=visname, refant=refant, config=args['config'], warn=False)
    File "/idia/software/pipelines/master/processMeerKAT/read_ms.py", line 177, in check_refant
        raise ValueError(err)
    ValueError: Reference antenna 'm059' isn't present in input dataset '1525469431_sdp_l0_2025-09-03T09-10-19_s1f.1025.4545454545455~1098.1818181818182MHz.mms'. Antennas present are: ['m001', 'm002', 'm003', 'm006', 'm008', 'm015', 'm021', 'm022', 'm034', 'm041', 'm042', 'm050', 'm051', 'm052', 'm060', 'm061']. Try 'm052' or 'm005' if present, or ensure 'calcrefant=True' and 'calc_refant.py' script present in '.config.tmp'.

    srun: error: compute-203: task 0: Exited with exit code 1
    srun: Terminating StepId=11534154.0
    ```
    In this case, the error message indicates a missing antenna — an issue we addressed earlier in the main tutorial. In other instances, the failure might occur during the `flag_round_1` or any other processing step. In such cases, you should inspect the log files for that step to identify and resolve the problem.


### Self-Calibration and Scince Image With ProcessMeerKAT

In this section we generate the config file to do Cross-calibration, Self-calibration and Scince Imaging at one go with ProcessMeerKAT. 

1. We Just have to follow the steps abouve and add some arguments when building the config.

    ```bash
    source /idia/software/pipelines/master/setup.sh
    processMeerKAT.py -B -C tutorial_config.txt -M /scratch3/users/walter/pipeline-training/1525469431/1525469431_sdp_l0_2025-09-03T09-10-19_s1f.ms -2 -I
    ```
    
2. Inspect the config and update where necessary. The new config should include these sections for selfcal and science imaging. Similarly you can refer to [DEEP2 tut](https://idia-pipelines.github.io/docs/processMeerKAT/config-files/) for detailed breakdown of the parameters.

    ```
    [selfcal]
    nloops = 2                        # Number of clean + bdsf loops.
    loop = 0                          # If nonzero, adds this number to nloops to name images or continue previous run
    cell = '1.5arcsec'
    robust = -0.5
    imsize = [6144, 6144]
    wprojplanes = 512
    niter = [10000, 50000, 50000]
    threshold = ['0.5mJy', 10, 10]    # After loop 0, S/N values if >= 1.0, otherwise Jy
    nterms = 2                        # Number of taylor terms
    gridder = 'wproject'
    deconvolver = 'mtmfs'
    calmode = ['','p']                # '' to skip solving (will also exclude mask for this loop), 'p' for phase-only and 'ap' for amplitude and phase
    solint = ['','1min']
    uvrange = ''                      # uv range cutoff for gaincal
    flag = True                       # Flag residual column after selfcal?
    gaintype = 'G'                    # Use 'T' for polarisation on linear feeds (e.g. MeerKAT)
    discard_nloops = 0                # Discard this many selfcal solutions (e.g. from quick and dirty image) during subsequent loops (only considers when calmode !='')
    outlier_threshold = 0.0           # S/N values if >= 1.0, otherwise Jy
    outlier_radius = 0.0              # Radius in degrees for identifying outliers in RACS

    [image]
    cell = '1.5arcsec'
    robust = -0.5
    imsize = [6144, 6144]
    wprojplanes = 512
    niter = 50000
    threshold = 10                    # S/N value if >= 1.0 and rmsmap != '', otherwise Jy
    multiscale = [0, 5, 10, 15]
    nterms = 2                        # Number of taylor terms
    gridder = 'wproject'
    deconvolver = 'mtmfs'
    restoringbeam = ''
    stokes = 'I'
    pbthreshold = 0.1                 # Threshold below which to mask the PB for PB correction
    pbband = 'LBand'                  # Which band to use to generate the PB - one of "LBand" "SBand" or "UHF"
    mask = ''
    rmsmap = ''
    outlierfile = ''
    ```

3. Once you are happy with the config file `run the pipeline`.

    ```
    processMeerKAT.py -R -C tutorial_config.txt
    ./submit_pipeline.sh
    ```

Once all the imaging jobs have completed, you should see your first science image, as shown below. Take a moment to inspect it carefully — does it meet your expectations? Consider what aspects could be improved, such as image quality, noise levels, or calibration accuracy. 

![ACT-CL J2023.3-5535 in L band](1525469431_sdp_l0_2025-09-03T09-10-19_s1f.ACT-CLJ2023.3-5535.science_image.image.tt0-image-2025-09-17-18-24-06.png)


# OXKAT Tutorial
