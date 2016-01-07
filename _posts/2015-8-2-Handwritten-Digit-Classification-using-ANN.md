---
layout: post
title: Handwritten digit classification using an ANN
---

Using Matlab to build from scratch, train and test an ANN that classifies handwritten digits.
No open source libraries are used. Everything is developed from scratch.

The dataset consists of 26 columns. The first containing the digit and the following 25
containing pixel intensities that represent the digit.

## Splitting into Train and Test Sets
First we divide our data into test and train sets.

```Matlab
train = csvread('train.csv');
subtrain= train(1:50000,:);
subtest= train(50001:60000,:);
```
The first 50,000 rows will be used to train the ANN and the final 10,000 will be used to evaluate its performance.

## The Neural Network

```Matlab
%% Neural Network
% Name: neuralnet.m
% Inputs: train_data = training set
%         test_data = test set
%         structure = structure of the neural network
%                     structure must have 25 as first value and 10 as last value
%                     eg: [25 100 10]

function predictions = neuralnet(train_data,test_data,structure)

numberOfLayers = length(structure);

%Random bias and weight values needed
biases=cell(1,numberOfLayers);
weights=cell(1,numberOfLayers);

%Creating random bias values
    for h=2:numberOfLayers
        biases{h} = zeros(structure(h) ,1);
    end
%Creating random weight values
    for h=2:numberOfLayers
        weights{h} = unifrnd(-1,1,structure(h),structure(h-1));
    end

%% Performing Stochastic Gradient Descent

%set the number of iterations to be performed
iterations = 2000;

    section_size=2000;
    loss = zeros(iterations,1) ;
    test_error = NaN(iterations,1) ;
    for m=1:iterations

           %pick random sample from the data
           index = randsample(1:length(train_data), section_size);
           samples = train_data(index,:);

           %%  Implement feedforward to get outputs pre-final layer.
           [beforelastlayer, node_datainputs]  = feedforward(weights,biases,samples,section_size,structure);
           pre_final_layer = beforelastlayer;

            %Push pre_final_layer through last layer activation function
            pre_activation =   pre_final_layer * weights{length(weights)}' + repmat(biases{length(biases)}',section_size,1);

            for i=1:section_size
                post_activation(i,:)= exp(pre_activation(i,:)) * 1/sum(exp(pre_activation(i,:))) ;

            end

            % Create matrix of true outputs
            true_output = zeros(length(samples),10);
            lossVector = zeros(length(samples),1);

            %calculating final loss
            for i=1:length(samples)
                true_output(i,samples(i,1)+1) = 1;

                lossVector(i,:) = -true_output(i,:) * log(post_activation(i,:)');

            end

             %Final Loss
             loss(m) = mean(lossVector) ;







            %%  Back Propagation algorithm

            %Final Layer delta
            delta_f = post_activation - true_output;

            %Delta's of layers in hidden layers
            hidden_d = hidden_deltas(weights,biases,structure, delta_f,node_datainputs,section_size);



            %%% implementing gradient descent with weight and bias updates

            weight_changeavg = cell(length(structure),1);
            bias_changeavg = cell(length(structure),1);

            learning_decay = 0.000001;

            %Learning rate decay
            learning_rate = 0.0001   -  m *  learning_decay * 0.000003;

            for l=length(structure):(-1):2
                weight_changeavg = node_datainputs{l}' * hidden_d{l} ;
                bias_changeavg = mean(hidden_d{l},1);


                %value updates

                weights{l} = weights{l} - learning_rate * weight_changeavg'  - 0.0000013 * sign(weights{l});
                biases{l} = biases{l} - learning_rate * bias_changeavg' ;

            end



     disp(m)


      %% Feeding into test data and getting loss.
     if rem(m,100)==0
       % Implement feedforward to get outputs pre-final layer.
           [beforelastlayer, node_datainputs]  = feedforward(weights,biases,test_data,length(test_data),structure);
           pre_final_layer = beforelastlayer;

            %Push pre_final_layer through last layer activation
            pre_activation =   pre_final_layer * weights{length(weights)}' + repmat(biases{length(biases)}',length(test_data),1);

            post_activation_test = zeros(length(test_data),structure(length(structure))) ;
            for i=1:length(test_data)
                post_activation_test(i,:)= exp(pre_activation(i,:)) * 1/sum(exp(pre_activation(i,:))) ;

            end


            true_output_test = zeros(length(test_data),10);
            lossVector_test = zeros(length(test_data),1);


         %and calculating final loss
                for i=1:length(test_data)
                    true_output_test(i,test_data(i,1)+1) = 1;

                    lossVector_test(i,:) = -true_output_test(i,:) * log(post_activation_test(i,:)');

                end

                 %Final Loss
                 test_error(m) = mean(lossVector_test) ;
     end



    end


    %% Plotting

    figure('Name','Loss on training data  ')
   plot(loss)
   title('Loss on training Data')
   xlabel('Iterations')
   ylabel('Loss')


  figure('Name', 'Error on evaluation data')
    plot(test_error,'o')
   title('Loss on evaluation Data')
   xlabel('Iterations')
   ylabel('Loss')



%% Calculate percentage error on test set.
[val,idx]= max(post_activation_test,[],2);


percentage_correct = sum((idx-1) - test_data(:,1) ==0) * 100 / (length(test_data) );


 disp(percentage_correct)

predictions = idx -1;


end

```
There are two other functions that are called by neuralnet.m

## Calculating the deltas
Calculates the deltas for each layer of the network

```Matlab
function output = hidden_deltas(weights,biases,structure, delta_f,node_datainputs,section_size)
%Computes the deltas for each layer.

% Place final layer's delta in last element of cell
all_deltas = cell(1,length(structure)) ;
all_deltas{length(all_deltas)} = delta_f;


    % Calculate intermediate values of deltas
    for l=(length(structure)-1):(-1):2
        all_deltas{l} = (weights{l+1}' * all_deltas{l+1}')' .* f_gradient(node_datainputs{l} * weights{l}' + repmat((biases{l})',section_size,1));


    end


output = all_deltas;

end

```

## Feedforward
Pushes the data through the neural network using a hyperbolic tangent activation.

```Matlab
% This takes in the inputs and feeds it through the network.
% Outputs a vector that contains the outputs of each sample fed through the network

function [final_output, node_datainputs]  = feedforward(weights,biases,a,section_size,structure)

% contains all observations in this batch


pre_activ = a(:,2:26);
node_datainputs=cell(1,length(structure));
node_datainputs{2}=pre_activ;

    for l = 2:(length(weights)-1)
        pre_activ = pre_activ * weights{l}' + repmat((biases{l})',section_size,1);
        % post_activ = tanh(pre_activ) ;
        node_datainputs{l+1} = tanh(pre_activ);

    end
final_output = tanh(pre_activ);
end

```

## Results
We can see that as the neural network is fed more data it gets better at recognizing the characters.

#### Loss on the training set
![Training Loss]({{ site.url }}_images/train_loss.jpg)


#### Loss on the test set
![Training Loss]({{ site.url }}_images/test_loss.jpg)
