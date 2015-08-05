---
layout: post
title: Handwritten digit classification using an ANN
---

Using Matlab to build from scratch, train and test an ANN that classifies handwritten digits.
No open source libraries are used. Everything is developed from scratch.

The dataset consists of 26 columns. The first containing the digit and the following 25
containing pixel intensities that represent the digit.

First we divide our data into test and train sets.

```Matlab
train = csvread('train.csv');
subtrain= train(1:50000,:);
subtest= train(50001:60000,:);
```
The first 50,000 rows will be used to train the ANN and the final 10,000 will be used to evaluate its performance.

```Matlab
function predictions = neuralnet(train_data,test_data,structure)

% structure must have 25 as first value and 10 as last value
%structure of NN eg: [25 100 10]
% sections = number of sections to divide up training data

no_layers = length(structure);

%Random bias and weight values needed
biases=cell(1,no_layers);
weights=cell(1,no_layers);

%Creating random bias values
    for h=2:no_layers
        biases{h} = zeros(structure(h) ,1);
    end
%Creating random weight values
    for h=2:no_layers
        weights{h} = unifrnd(-1,1,structure(h),structure(h-1));
    end

%% Performing Stochastic Gradient Descent

iterations = 1000;


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

            %Push pre_final_layer through last layer activation
            pre_activation =   pre_final_layer * weights{length(weights)}' + repmat(biases{length(biases)}',section_size,1);

            for i=1:section_size
                post_activation(i,:)= exp(pre_activation(i,:)) * 1/sum(exp(pre_activation(i,:))) ;

            end




            % Create matrix of true outputs
            true_output = zeros(length(samples),10);



            loss_vec = zeros(length(samples),1);

            %and calculating final loss
            for i=1:length(samples)
                true_output(i,samples(i,1)+1) = 1;

                loss_vec(i,:) = -true_output(i,:) * log(post_activation(i,:)');

            end

             %Final Loss
             loss(m) = mean(loss_vec) ;







            %%  BackProp algorithm

            %Final Layer delta
            delta_f = post_activation - true_output;

            %Delta's of layers in hidden layers
            hidden_d = hidden_deltas(weights,biases,structure, delta_f,node_datainputs,section_size);



            %%% implementing gradient descent / weight and bias updates

            weight_changeavg = cell(length(structure),1);
            bias_changeavg = cell(length(structure),1);

            learning_decay = 0.000001;

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
            loss_vec_test = zeros(length(test_data),1);


         %and calculating final loss
                for i=1:length(test_data)
                    true_output_test(i,test_data(i,1)+1) = 1;

                    loss_vec_test(i,:) = -true_output_test(i,:) * log(post_activation_test(i,:)');

                end

                 %Final Loss
                 test_error(m) = mean(loss_vec_test) ;
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
