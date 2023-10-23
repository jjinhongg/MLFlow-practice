# Example MLFlow Projects

This repository is part of the MLOps Coursera Course with practical examples to apply different MLFlow techniques for understanding the components of MLFlow better.


## Create an MLFlow Project

Find more about what it takes to create an MLFlow project with Conda, a packaging solution that can work with Python as well as other system dependencies.

For this project, we'll use a small CSV file with data about wines, called [wine-ratings.csv](wine-ratings.csv).

### Step 1: Start with the dependencies

Before going into the details of the project, you probably have a good idea of what you need to install. I recommend setting up a Conda environment first. Even if you prefer using `pip` for installing dependencies, you can still define that with a Conda environment.

This early in the project's lifecycle you are probably not ready to do some training, so let's only concentrate in exploratory work for the dataset.

```
conda create --name exploratory python=3.8
```

Once the environment is created, activate it so that packages are installed in that activated environment.

```
conda activate exploratory
```

Next, with the _exploratory_ environment activated export the dependencies so a new YAML file named _conda_env.yml_

```
conda env export --name exploratory > conda_env.yml
```

You should end up with a file similar to this:

```yaml
name: exploratory
channels:
  - conda-forge
dependencies:
  - bzip2=1.0.8
  - ca-certificates
  - libffi=3.4.2
  - libsqlite=3.39.3
  - libzlib=1.2.12
  - ncurses=6.3
  - openssl=3.0.5
  - python=3.8.13
  - readline=8.1.2
  - setuptools=65.3.0
  - sqlite=3.39.3
  - tk=8.6.12
  - wheel=0.37.1
  - xz=5.2.6
  - pip
```

Next, append a few lines to use `pip` to install `mlflow` and `pandas`. This is not a hard requirement, but it is useful to know that you can use the conda environment file to add more dependencies using `pip` and not `conda`. The newly appended lines should look like this:

```yaml
  - pip:
    - pandas==1.5.0
    - mlflow==2.7.1
```

The full file should look exactly like [exploratory/conda_env.yml](./exploratory/conda_env.yml)

Since you added the `pip` installs directly to the file, you didn't really install them, so go ahead and run the following command to ensure everything is working:

```
 conda env update --file conda_env.yml --prune
```

The command will look into any new (or different) dependencies and install them. The `--prune` flag will remove anything that is no longer defined in the _conda_env.yml_ file. In this case, you didn't remove anything, but it is still a good idea to keep using it.

Also do a `pip install -r requirements.txt` to ensure you have all the libraries required for the subsequent sections.

## Step 2: MLFlow project file
The project name will be the name of the directory. In this case, our project will be `exploratory` that we are going to use from the `/exploratory` directory.

You will need to specify options by adding a `MLproject` file, which is a `text` file with `YAML` syntax. An example is shown below:

```yaml
# An MLFlow project that has a single entry point to validate a data set
name: Dataset Validation

conda_env: conda_env.yml

entry_points:

  main:
    parameters:
      log: {type: bool, default: true}
      max_errors: {type: int, default: 1}
      filename: path
    command: "python validate.py {log} {max_errors} {filename}"

```

## Step 3: Run the project
Run:
```
mlflow run . -P filename=carriage.csv
```
**Arguments**
- `.`: Specifies the current directory
- `-P`: Pass arguments
- `filename`: The file path that contains the sample data.

### Run Project from a remote Git repo
An example of how to run from a Git repo:
```
mlflow run https://github.com/mlflow/mlflow-example.git -P alpha=0.4
```
MLproject file has to be in the `mlflow/mlflow-example` directory.

# Serving a MLFlow model
In this section we will be focusing on the `/serve` and `register` directory. Follow the steps in this ['models.ipynb'](/workspaces/MLFlow-practice/register/models.ipynb) to run the `log_model.py` script. The notebook will provide an idea on how to update model versions, descriptions and change the name if necessary.

## Serve
```bash
cd serve
ls mlruns/0
```
We should see something similar to the this:
```
a50e811ab82646f790e652932f330afd  meta.yaml
```
and if we look at the tree structure using:
```
tree mlruns/0/a50e811ab82646f790e652932f330afd/artifacts
```
We will see the something similar to this:
```
mlruns/0/a50e811ab82646f790e652932f330afd/artifacts
└── model
    ├── MLmodel
    ├── conda.yaml
    ├── model.pkl
    ├── python_env.yaml
    └── requirements.txt
```
From the tree structure, under a model run (a string of gibberish characters), we can observe the storage format of a model. Over here for the model `model`, the artifacts listed as: 
- `model.pkl`: The actual model in `.pkl`
- `conda.yaml`: The conda environment `.yml`
- `python_env.yaml`: The python environment
- `requirements.txt`: Packages that are used for the model run

When serving the model, it will look for these artifacts to understand how to serve the model. It will also look into their respective `$PATH`. So, as we have been using `conda` and in case we do not have the python environment in our `$PATH`, run the code below in the bash terminal:
```
curl https://pyenv.run | bash
python -m  pip install virtualenv
PATH="$HOME/.pyenv/bin:$PATH"
```

Now, we are ready to serve the model by running this code in our terminal:
```
mlflow models serve -m runs:/a50e811ab82646f790e652932f330afd/model -p 5001
```
You can use any port that are not in use. We are using -p 5001 since 5000 was in use for the localhost server.

Example of how to perform predictions with the served model:
```bash
curl -X POST -H "Content-Type:application/json; format=pandas-split" --data '{"columns":["text"],"data":[["Today is a perfect day to practice automation skills"]]}' http://127.0.0.1:5000/invocations
```
