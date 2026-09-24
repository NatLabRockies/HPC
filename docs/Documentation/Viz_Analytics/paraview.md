# ParaView

*ParaView is an open-source, multi-platform data analysis and visualization application. ParaView users can quickly build visualizations to analyze their data using qualitative and quantitative techniques. The data exploration can be done interactively in 3D or programmatically using ParaView's batch processing capabilities. ParaView was developed to analyze extremely large data sets using distributed memory computing resources. It can be run on supercomputers to analyze data sets of terascale as well as on laptops for smaller data.*

The following tutorials are meant for Kestrel supercomputer. 


## Using ParaView in Client-Server Mode 

Running ParaView in client-server mode is a convenient workflow for researchers who have a large amount of remotely-stored data that they'd like to visualize using a locally-installed copy of ParaView. 

In this model, the HPC does the I/O and computational work, then "serves" the rendered data to your local ParaView client. This means all of your preferences and shortcuts on your local client work without moving data from the HPC or using a remote desktop enviroment.

!!! note "Client Installation"
    It is required that you use binaries provided by Kitware locally that match the NLR installed version for compatibility.
    
    To determine which version of ParaView is installed on the cluster, connect to Kestrel, load the ParaView module with `module load paraview`, then check the version with `pvserver --version`.
    
    Download the ParaView client binary which matches the version on Kestrel on the [ParaView website](https://www.paraview.org/download/). 

### Steps for Connecting to Kestrel with ParaView
1. Reserve Compute Nodes

    Step 1 is to reserve the computational resources on Kestrel that will be running the ParaView server.
    
    This requires using the Slurm `salloc` directive and specifying an allocation name and time limit for the reservation.
    To reserve the computational resources:
    
    ```bash
    salloc -A <allocation> -t <time_limit>
    ```
    
    where `<allocation>` will be replaced with the allocation name you wish to charge your time to and `<time_limit>` is the amount of time you're reserving the nodes for. This requests a single node that we can launch a maximum number of 104 ParaView server processes on in Step 2.

    Note the name of the node that the Slurm scheduler assigns you (it is what follows your username and "@" symbol where you input text, e.g., x1008c0s0b1n1) as we will need it in Step 3.
    
    To launch more than 104 ParaView server processes, you will need to request multiple nodes:

    ```bash
    salloc -A <allocation> -t <time_limit> -N 2
    ```
    
    where the `-N 2` option specifies that two nodes be reserved, which means the maximum number of ParaView servers that can be launched in Step 2 is  104 x 2 = 208. Note that even with multiple nodes, the only server name to copy for Step 3 is still the one immediately following the "@" symbol.

2. Start ParaView Server

    After reserving the compute nodes, load the ParaView module with:
    
    ```bash
    module load paraview
    ```
    
    Next, start the ParaView server with a call to the Slurm `srun` directive:
    
    ```bash
    srun -n 8 pvserver --force-offscreen-rendering
    ```
    
    In this example, the ParaView server will be started on 8 processes.  

    !!! note "Headless Rendering"
        The `--force-offscreen-rendering` option is present to ensure that, where possible, CPU-intensive filters and rendering calculations will be performed server-side (i.e., on the Kestrel compute nodes) and *not* on your local machine.
    
    
    Although every dataset may be different, ParaView offers the following recommendations for balancing grid cells to processors:
    
    | Grid Type         | Target Cells/Process | Max Cells/Process |
    | ----------------- | -------------------- | ----------------- |
    | Structured Data   | 5-10 M               | 20 M              |
    | Unstructured Data | 250-500 K            | 1 M               |
    
    For example, if you have data stored in an unstructured mesh with 6 M cells, you'd want to aim for between 12 and 24 ParaView server processes, which easily fits on a single Kestrel node.  
    
    If the number of unstructured mesh cells was instead around 60 M, you'd want to aim for 120 to 240 processes, which means requesting a minimum of 2 Kestrel nodes.  
    
    Note that this 2-nodes request may remain in the queue longer while the scheduler looks for resources, so depending on your needs, it may be necessary to factor queue times into your optimal cells-per-process calculation.
    
    !!! note "Port Selection"
        The `--server-port=<port>` option may be used with pvserver if you wish to use a port other than 11111 for Paraview. You will need to adjust the port in the SSH tunnel and tell your Paraview client which port to use, as well. 
        
        See the following sections for details.

3. Create SSH Tunnel

    Next, create an SSH tunnel to connect your local desktop to the compute node(s) you reserved in Step 1.   
    
    If you have changed the port via the `--server-port=<port>` flag, you must change the above command from the default port 11111 to your selected port.
    
    Open a new local terminal window:
    
    ```bash
    ssh -L 11111:<node_name>:11111 <username>@kestrel.hpc.nlr.gov
    ```
    
    where `<node_name>` is the node name you copied in Step 1 and `<username>` is your HPC username.


4. Connect ParaView Client

    Now that the ParaView server is running on a compute node and your desktop is connected via the SSH tunnel, you can open ParaView as usual.  
    From here, click the "Connect" icon or `File > Connect`.  
    
    Next, click the "Add Server" button and enter the following information. Again, note that if you changed the port before, you must reflect that change here.
    
    | Name        | Value         |
    |-------------|---------------|
    | Name        | Kestrel HPC   |
    | Server Type | Client/Server |
    | Host        | localhost     |
    | Port        | 11111         |
    
    Only the last three fields, Server Type, Host, and Port, are strictly necessary (and many of them will appear by default) while the Name field can be any recognizable string you wish to associate with this connection.  
    
    When these 4 fields have been entered, click "Configure" to move to the next screen, where we'll leave the Startup Type set to "Manual".  
    
    !!! note "Subsequent Connections"
        While you will need to still perform the first 3 steps, once you have performed Step 4 once and saved, you may simply double-click on the saved connection every following time.

    When finished, select the server just created and click "Connect".  
    
    The simplest way to confirm that the ParaView server is running as expected is to view the Memory Inspector toolbar (`View > Memory Inspector`) where you should see a ParaView server for each process started in Step 2 (e.g., if `-n 8` was specified, processes `0-7` should be visible).
    
    You can now `File > Open` your data files as you normally would, but instead of your local hard drive you'll be presented with a list of the files stored on Kestrel.

### General Tips

* The amount of time you can spend in a post-processing session is limited by the time limit specified when reserving the compute nodes in Step 1.  If saving a large time series to a video file, your reservation time may expire before the video is finished.  Keep this in mind and make sure you reserve the nodes long enough to complete your job.
* Adding more parallel processes in Step 2, e.g., `-n 36`, doesn't necessarily mean you'll be splitting the data into 36 blocks for each operation.  ParaView has the *capability* to use 36 parallel processes, but may use many fewer as it calculates the right balance between computational power and the additional overhead of communication between processors.







## High-quality Rendering With ParaView 

How to use ParaView in batch mode to generate single frames and animations on Kestrel

###  Building PvBatch Scripts in Interactive Environments

1.  Begin by connecting to a Kestrel login node:

    ```bash
    ssh <username>@kestrel.hpc.nlr.gov
    ```

2.  Request an interactive compute session for 60 minutes:

    ```bash
    salloc -A <allocation> -t 60
    ```

3.  Once the session starts, load the appropriate modules:

    ```bash
    module purge
    module load paraview/5.11.0-server
    ```
    
    !!! note "paraview/5.11.0-server for Offscreen Rendering"
        In this case, we select the `paraview/5.11.0-server` module as opposed to the default ParaView build, as the server version is built for rendering using offscreen methods suitable for compute nodes.

4.  Start your render job:

    ```bash
    srun -n 1 pvbatch --force-offscreen-rendering render_sphere.py
    ```

    where `render_sphere.py` is a simple ParaView Python script to add a sphere source and save an image.

###  Transitioning to Batch Post-Processing

Tweaking the visualization options contained in the `pvrender.py` file inevitably requires some amount of trial and error and is most easily accomplished in an interactive compute session like the one outlined above.  Once you feel that your script is sufficiently automated, you can start submitting batch jobs that require no user interaction.

1.  Prepare your script for `sbatch`. A minimal example of a batch script named `batch_render.sh` could look like:

    ```bash
    #!/bin/bash

    #SBATCH --account=<allocation>
    #SBATCH --time=60:00
    #SBATCH --job-name=pvrender
    #SBATCH --nodes=2

    module purge
    module load paraview/$version-server
    
    srun -n 1 pvbatch --force-offscreen-rendering render_sphere.py 1 &
    srun -n 1 pvbatch --force-offscreen-rendering render_sphere.py 2 &
    srun -n 1 pvbatch --force-offscreen-rendering render_sphere.py 3 &

    wait
    ```

    where we run multiple instances of our dummy sphere example, highlighting that different options can be passed to each to post-process a large batch of simulated results on a single node.  Note also that for more computationally intensive rendering or larger file sizes (e.g., tens of millions of cells) the option `-n 1` option can be set as suggested in the [client-server guide](./paraview.md#using-paraview-in-client-server-mode).


2.  Submit the job and wait:

    ```
    sbatch batch_render.sh
    ```


###  Tips on Creating the PvBatch Python Script

The easiest way to create your ParaView Python script is to run a fresh session of ParaView (use version 5.x on your local machine) and select "Tools → Start Trace," then "OK". Perform all the actions you need to set your scene and save a screenshot. Then select "Tools → Stop Trace" and save the resulting python script (we will use `render_sphere.py` in these examples).
 

Here are some useful components to add to your ParaView Python script:

-   Read the first command-line argument and use it to select a data file to operate on.

    ```python
    import sys
    doframe = 0
    if len(sys.argv) > 1:
        doframe = int(sys.argv[1])
    infile = "output%05d.dat" % doframe
    ```

    !!! note "Individual Frame Rendering"
        Note that `pvbatch` will pass any arguments after the script name to the script itself, so you can do the following to render frame 45:
        ```bash
        srun -n 1 pvbatch --force-offscreen-rendering render_sphere.py 45
        ```
        You could programmatically change this value inside the `batch_render.sh` script, your script would need to iterate using something like:
        ```bash
        for frame in 45 46 47 48
        do
            srun -n 1 pvbatch --force-offscreen-rendering render_sphere.py $frame
        done
        ```

<!--     And you would need to submit the script as such:

        sbatch -F "45" batchrender.sh -->

-   Set the output image size to match FHD or UHD standards:

    ```python
    renderView1.ViewSize = [3840, 2160]
    renderView1.ViewSize = [1920, 1080]
    ```

-   Don't forget to actually render the image!

    ```python
    pngname = "image%05d.png" % doframe
    SaveScreenshot(pngname, renderView1)
    ```

## Insight Center

ParaView is supported in the Insight Center's immersive virtual environment. 
[Learn about the Insight Center](https://www.nlr.gov/computational-science/insight-center.html). 

For assistance, contact [Kenny Gruchalla](Kenny.Gruchalla@nlr.gov).
