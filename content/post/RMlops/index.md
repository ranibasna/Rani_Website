---
date: "2022-10-12"
diagram: true
tags: [Automation, Google cloud platform, Stock prediction, time series]
categories: [R, cloud build, GitHub actions, Docker]
highlight: true
links:
- icon: github-square
  icon_pack: fab
  name: GitHub
  url: https://github.com/ranibasna/RMlops
image:
  caption: ''
  placement: 3
math: true
title: Machine learning operations for R
---



In this article, I will present a one effective way to implement Mlops approach while using R. The approach that I will be using incorporate GitHub actions, Docker, Google cloud platform, and R code.

## The data

For the purpose of this article, I will be using Apple stock price data. I will fetch the data on a daily basis. The objective here is to build a model that can predict the direction of the stock for on a weakly basis. That each weeak the model will output a new prediction on the direction of the asset price.

## The Model

For the purpose of this article,  I will not go after a perfect prediction model as our attention her is to show how can we use R to conduct Mlops scenario. Once the data is processed and ready for modeling, I will use xgboost R package to fit the model. I will also be using tidymdoel framework for developing the model within R.

## Mlops pipline and steps

Here is a short description of the workflow steps for this project.

 1. Write a code for extract and processing the data. The resulted data should be ready to feed the model.
 2. Use GitHub actions to automate the extraction code of the data on a daily basis.
 3. Write a code to train, test, and validate the xgboost model.
 4. Use GitHub actions to automate the model training on a weakly basis.
 5. Write an R code to preform a prediction form the trained saved model.
 5. Write a Dockerfile to containerize the prediction of the model
 6. Use the plumber package to serve the model as api.
 7. Use the cloud build service from Google cloud platform to deploy the api on the cloud.
 8. Use automated triggeres from cloud build to automate the build on changes from the model or pushes to the GitHub repo.



## Yaml file for the parameters

Here we will define a set of the parameters for the input values,  output paths, the model and the data extraction. We will call the parameters value within the R files.

```
data_extraction:
  first_date : 2007-01-01
  output_path : data
  output_filename_data : StockData.RData

train:
  input_data: data/StockData.RData
  initial: 600
  assess: 50
  model_mode: regression
  n_trees: 200
  model_engin: xgboost
  grid_size: 20
  best_model_output: outputs/best_model.RData
  training_data_output: outputs/training_data.RData

predict:
  model_path: outputs/best_model.RData
```

## R code

### Data extraction code

Here we will write two R files. The first one is to extract the data using the tidyquant package. Next we call the related parameters values. Using the tidyquant we fetch Apple stock prices for the specific date. Moreover, we apply some data processing to insure that we have a weakly prices and returns. Finally, we calculate some of the important indicator that is use to approximate the direction of the stock. Here, we will be using the Moving Average Convergence Divergence (MACD), the Relative strength indicator (RSI), and the simple moving average (MV).

```{r libraries}
libs = c('tidyquant', 'quantmod', 'lubridate', 'dplyr', 'yaml', 'glue')

sapply(libs[!libs %in% installed.packages()], install.packages)
sapply(libs, require, character.only = T)

# Read parameters file
parameters = read_yaml('configuration/parameters.yaml')[['data_extraction']]

first_date = parameters$first_date
output_path = parameters$output_path
output_filename_data = parameters$output_filename_data

# Create full paths
output_data = glue("{output_path}/{output_filename_data}")

# get the data on daily basis
mystock <- tq_get("AAPL", from = first_date, to = Sys.Date())
# convert to weekly data prices
mystock_weeks <- mystock %>%  tq_transmute(select= open:adjusted, mutate_fun = to.period, period = "weeks")
# calculate the weekly returns
mystock_weeks_ret <- mystock_weeks %>% tq_mutate(select = adjusted, mutate_fun = periodReturn, period = "weekly", type = "arithmetic")
# add the indicators MACD, RSI, and Simple moving average 20,50,10
mystock_weeks_ret_ind <- mystock_weeks_ret %>% tq_mutate(select= close, mutate_fun = MACD, nFast= 12, nSlow= 26, nSig= 9, maType= SMA) %>% mutate(diff = macd - signal) %>% tq_mutate(select = close, mutate_fun = RSI, n =14) %>% tq_mutate(select = close, mutate_fun = SMA, n=20) %>% tq_mutate(select = close, mutate_fun = SMA, n=50) %>% tq_mutate(select = close, mutate_fun = SMA, n=150) %>% dplyr::mutate(direction = ifelse(weekly.returns > 0, 1, 0)) %>% tidyr::drop_na()

# data processing
mystock_final_data <- mystock_weeks_ret_ind %>% select(-c(open, high, low))
# lagg the data using lead
mystock_final_data_laged <- mystock_final_data %>% mutate(close_lead = lead(close), date_lead = lead(date)) %>% select(-c(date, close, direction)) %>% tidyr::drop_na()

# Save the file
saveRDS(mystock_final_data_laged, output_data)
```

### Model training script

In here we are using the xgboost algorithm to model our time series data. For the sake of training the data we follow roling origin approach as opposed to cross validation using rsample package. In reality, this step takes plenty if time and integration. The best approach her is to use a special Mlops platform to logg the model changes. Running different scenarios and methods are encourged and goes more smooth by such software. For instance, one can use Mlflow, Neptunai, and many others. Hoever, here for the sake of simplicity I will use the final model and leave the model logging step for another blog. Simillarly, I will not go into explaining the reasoning behind the model and the steps that used to develop this model. Different R packages are available here. My preferred approach is the tidymodels workflow.

```{r}
libs = c('yaml', 'dplyr', 'timetk', 'lubridate','xgboost','tidymodels','tidyquant')

sapply(libs[!libs %in% installed.packages()], install.packages)
sapply(libs, require, character.only = T)

# Load Variables
train_data_info =  read_yaml('configuration/parameters.yaml')[['train']]
input = train_data_info$input_data
initial = train_data_info$initial
assess = train_data_info$assess
model_mode = train_data_info$model_mode
n_trees = train_data_info$n_trees
model_engin = train_data_info$model_engin
grid_size = train_data_info$grid_size
best_model_output = train_data_info$best_model_output
training_data_output = train_data_info$training_data_output

# Read Data
stock_data = readRDS(input)
# neptune_api_key = Sys.getenv('api_key')

# data spliting with rolling origin using rsample package
rolling_df <- rsample::rolling_origin(stock_data, initial = initial, assess = assess, cumulative = FALSE, skip = 0)

# XGBoost model specification
xgboost_model <-
  parsnip::boost_tree(
    mode = model_mode,
    trees = n_trees,
    min_n = tune(),
    tree_depth = tune(),
    learn_rate = tune(),
    loss_reduction = tune()
  ) %>% set_engine(model_engin, objective = "reg:squarederror")

# grid specification
xgboost_params <-
  dials::parameters(
    min_n(),
    tree_depth(),
    learn_rate(),
    loss_reduction()
  )
#
xgboost_grid <-
  dials::grid_max_entropy(
    xgboost_params,
    size = grid_size
  )

# set workflow
xgboost_wf <-
  workflows::workflow() %>%
  add_model(xgboost_model) %>%
  add_formula(weekly.returns ~ .)


# hyperparameter tuning
xgboost_tuned <- tune::tune_grid(
  object = xgboost_wf,
  resamples = rolling_df,
  grid = xgboost_grid,
  metrics = yardstick::metric_set(rmse, rsq, mae),
  control = tune::control_grid(verbose = TRUE)
)

#
xgboost_best_params <- xgboost_tuned %>% tune::select_best("rmse")
xgboost_model_best <- xgboost_model %>%  finalize_model(xgboost_best_params)

# split into training and testing data sets. stratify by weekly return
data_split <- stock_data %>%  timetk::time_series_split(initial = dim(stock_data)[1] - 60, assess = 60)

training_df <- training(data_split)
test_df <- testing(data_split)

test_prediction <- xgboost_model_best %>%
  # fit the model on all the training data
  fit(
    formula = weekly.returns ~ .,
    data    = training_df
  ) %>%
  # use the training model fit to predict the test data
  predict(new_data = test_df) %>%
  bind_cols(testing(data_split))

# measure the accuracy of our model using `yardstick`
xgboost_score <- test_prediction %>% yardstick::metrics(weekly.returns, .pred) %>% mutate(.estimate = format(round(.estimate, 2), big.mark = ","))

# Save the fitted version of the best model
xgboost_model_best <- xgboost_model_best %>% fit(formula = weekly.returns ~ ., data    = training_df)
saveRDS(xgboost_model_best, best_model_output)
```

## Plumber Api

From their website: Plumber allows you to create a web API by merely decorating your existing R source code with roxygen2-like comments. The plumber R package allows users to expose existing R code as a service available to others on the Web. It convert a function into an API in a very simple way. We simply have to indicate the parameters of our function (which is at, the same time, the parameters of the API) and the type of request to make. The value returned by our function will be precisely the value returned by our API.

### Model prediction

```{r}
libs = c('yaml', 'tidyverse','tidymodels','tidyquant','xgboost')

sapply(libs[!libs %in% installed.packages()], install.packages)
sapply(libs, require, character.only = T)

# Load Variables
predict_data_info =  read_yaml('configuration/parameters.yaml')[['predict']]
model_path = predict_data_info$model_path

#* @apiTitle stock prediction for the next week Apple stock direction
#* Charting Apple stock
#* @get /plot
#* @serializer contentType list(type='image/png')

function(){
  AAPL <- tq_get("AAPL", from = Sys.Date() - 1300, to = Sys.Date())
  plot <- ggplot(AAPL,aes(x=date, y=close,
                          open = open, high = high, low = low, close = close))+ geom_line(size=1) +
    geom_bbands(ma_fun = SMA, sd = 2, n = 30, linetype = 5)
  file <- "plot.png"
  ggsave(file, plot)
  readBin(file, "raw", n = file.info(file)$size)
}

#* Returns most recent week apple stock direction
#* @get /AppleSharePrice

function(){
  # get the data on daily basis
  new_data_input <- tq_get("AAPL", from = Sys.Date() - 1300, to = Sys.Date())
  # convert to weekly data prices
  new_data_input <- new_data_input %>%  tq_transmute(select= open:adjusted, mutate_fun = to.period, period = "weeks")
  # calculate the weekly returns
  new_data_input <- new_data_input %>% tq_mutate(select = adjusted, mutate_fun = periodReturn, period = "weekly", type = "arithmetic")
  # add the indicators MACD, RSI, and Simple moving average 20,50,10
  new_data_input <- new_data_input %>% tq_mutate(select= close, mutate_fun = MACD, nFast= 12, nSlow= 26, nSig= 9, maType= SMA) %>% mutate(diff = macd - signal) %>% tq_mutate(select = close, mutate_fun = RSI, n =14) %>% tq_mutate(select = close, mutate_fun = SMA, n=20) %>% tq_mutate(select = close, mutate_fun = SMA, n=50) %>% tq_mutate(select = close, mutate_fun = SMA, n=150) %>% dplyr::mutate(direction = ifelse(weekly.returns > 0, 1, 0)) %>% tidyr::drop_na()

  # data processing
  new_data_input <- new_data_input %>% select(-c(open, high, low))
  # lag the data using lead
  new_data_input <- new_data_input %>% mutate(close_lead = lead(close), date_lead = lead(date)) %>% select(-c(date, close)) %>% tidyr::drop_na()

  # filter last week data
  new_data_input_laged <- new_data_input %>% filter(date_lead > Sys.Date() - 14)

  # Load model
  xgboost_model_final = readRDS(model_path)

  # predictions
  predictions = xgboost_model_final  %>%  predict(new_data = new_data_input_laged)
  result <- list(predictions, new_data_input_laged$date_lead)
  return(result)
}
```

## Deploy the model on Google cloud

In this section, we will Dockerize our Plumber API. Then we will use google cloud to build a Docker image that can be deployed for prediction of our Apple stock direction.


### Docker file

Let us first write the Docker File.

```
FROM asachet/rocker-tidymodels

# Install required libraries
RUN R -e 'install.packages(c("plumber", "yaml", "dplyr", "tidymodels", "tidyquant","xgboost"))'

# Copy model and script
RUN mkdir /data
RUN mkdir /configuration
RUN mkdir /outputs
COPY configuration/parameters.yaml /configuration
COPY data/StockData.RData /data
COPY prediction_api.R /
COPY outputs/best_model.RData /outputs


# Plumb & run server
EXPOSE 8080
ENTRYPOINT ["R", "-e", \
    "pr <- plumber::plumb('prediction_api.R'); pr$run(host='0.0.0.0', port=8080)"]
```

### The cloudbuild file

Next,  we write the YAML cloudbuild file. This will build our docker image from the docker file. The build will only run after a trigger goes on.  For our specific use case, we will only build the docker image and deploy the model when there is a new model version available. In this scenario, this will be about once a week as new weekly data become available.

```
steps:
- name: 'gcr.io/cloud-builders/docker'
  args: ['build', '-t', 'gcr.io/RMLOps/github.com/https://github.com/ranibasna/RMlops:$SHORT_SHA', '.']
- name: 'gcr.io/cloud-builders/docker'
  args: ['push', 'gcr.io/RMLOps/github.com/https://github.com/ranibasna/RMlops:$SHORT_SHA']
- name: 'gcr.io/cloud-builders/gcloud'
  args: ['beta', 'run', 'deploy', 'RMLOps', '--image=gcr.io/RMLOps/github.com/https://github.com/ranibasna/RMlops:$SHORT_SHA', '--region=europe-west1', '--platform=managed']
```
## GitHub actions

GitHub Actions is a continuous integration and continuous delivery (CI/CD) platform that allows you to automate your build, test, and deployment pipeline. You can create workflows that build and test every pull request to your repository, or deploy merged pull requests to production. Each of your repositories gets 2,000 minutes of free testing per month.

GitHub Actions goes beyond just DevOps and lets you run workflows when other events happen in your repository. For example, you can run a workflow to automatically add the appropriate labels whenever someone creates a new issue in your repository.

Initially, Github Actions is intended to perform automation within Github in order to make certain tasks easier for developers.

However, as data scientists, Github Actions opens a very interesting door for us to automate issues such as data extraction or model retraining, among others, whether we work with Python, R or any other language.

### Automate data extraction
```
name: update_data

on:
  # schedule:
  #  - cron: '0 7 * * *' # Execute every day
  workflow_dispatch:

jobs:
  build:
    runs-on: ubuntu-20.04 # (ubuntu-latest)
    steps:
      - name: Checkout
        uses: actions/checkout@v2

      - name: Update
        run: sudo apt-get update

      - name: Install necesary libraries
        run:  sudo apt-get install -y curl libssl-dev libcurl4-openssl-dev libxml2-dev #sudo apt-get install -y r-cran-httr -d

      - name: Install R
        uses: r-lib/actions/setup-r@v2 # Instalo R
        with:
          r-version: '4.1.2'

      - name: Run Data Extraction
        run: Rscript Rcode/extract_daily_data.R Rcode

# Add new files in data folder, commit along with other modified files, push
      - name: Commit files
        run: |
          git config --local user.name actions-user
          git config --local user.email "actions@github.com"
          git add data/*
          git commit -am "GH ACTION Headlines $(date)"
          git push origin master
        env:
          REPO_KEY: ${{secrets.GITHUB_TOKEN}}
          username: github-actions
```

### Automate model retraining

```
name: retrain_model

on:
  repository_dispatch:
    types: retrain_model
  workflow_dispatch:

jobs:
  build:
    runs-on: ubuntu-20.04 # (ubuntu-latest)
    # container:
    #  image: asachet/rocker-tidymodels

    steps:
      - name: Checkout
        uses: actions/checkout@v2

      - name: Update
        run: sudo apt-get update

      - name: Install necesary libraries
        run:  sudo apt-get install -y curl libssl-dev libcurl4-openssl-dev libxml2-dev #sudo apt-get install -y r-cran-httr -d

      - name: Install R
        uses: r-lib/actions/setup-r@v2 # Instalo R
        with:
          r-version: '4.1.2'

      # - name: Create and populate .Renviron file
      #   run: |
      #     echo TOKEN="$secrets.TOKEN" >> ~/.Renviron
      #     echo api_key="$secrets.API_KEY" >> ~/.Renviron
      #
      # - name: Setup Neptune
      #   run: Rscript src/neptune_setup.R

      - name: Retrain Model
      #  env:
      #    NEPTUNE_API_TOKEN: ${{ secrets.API_KEY }}
        run: Rscript Rcode/model_training.R

      # Add new files in data folder, commit along with other modified files, push
      - name: Commit files
        run: |
          git config --local user.name actions-user
          git config --local user.email "actions@github.com"
          git add data/*
          git commit -am "GH ACTION Headlines $(date)"
          git push origin master
        env:
          REPO_KEY: ${{secrets.GITHUB_TOKEN}}
          username: github-actions
```
