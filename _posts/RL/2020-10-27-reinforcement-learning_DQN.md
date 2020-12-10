---
title : "Cube_RL_DQN"

date : 2020-10-27

categories : rl
---

## DQN에이전트를 사용하여 큐브가 목표지점 찾아가게하는 간단한 알고리즘

#### DQN의 등장


<img src = "/surabanke/assets/images/DQN_0.png" width = "1000">




<img src = "/surabanke/assets/images/table.png" width = "1000">




너무 큰 state-action value pair를 가진 환경은 table을 전부 메모리에 담을 수 없기에 value function approximator를 주어 함수형태로 표현이 가능하다. -> solution for large MDP

value function approximator(W) 파라미터는 뉴럴넷에서의 weight파라미터라고 보면된다.
GOAL : find parameter W minimising loss between approximate value and true value
실제값의 근사값을 얻는다면
1. 메모리 절약
2. 실제 data의 noise도 상관없다.
3. 실제 가지고 있지 않은 data도 function에 넣어 구할 수 있다.


<img src = "/surabanke/assets/images/DQN.png" width = "1000">


#### Cube DQN 구현코드

    classdef my_ENV < rl.env.MATLABEnvironment
    %MY_ENV: Template for defining custom environment in MATLAB.    

    %% Properties (set properties' attributes accordingly)

        % Specify and initialize environment's necessary properties    
        % Acceleration due to gravity in m/s^2
         properties


         MaxForce = 1;
        % action total number
         act_num = 0;
         %Reward_A = 0;

        % sample time
         Ts = 0.02;

        % Reward for the agent within range.
         Reward = 40;

        % Penalty for the agent if out of range.

         Penalty = -40;

        % Initialize the cube start place (initialize the state)
         %box = fill(State(1,:),State(2,:),'b');

        % States [x_c1 x_c2 x_c3 x_c4;y_c1 y_c2 y_c3 y_c4]
         State = zeros(2,4);
        end

        properties(Access = protected)
        % Initialize episode
         IsDone = false


        end

    %% Necessary Methods methods
    % Define state and action
        function this = my_ENV()

        ObservationInfo = rlNumericSpec([2 4]);
        ObservationInfo.Name = 'ENV_States';

        ActionInfo = rlFiniteSetSpec({[-1],[0],[1]});
        ActionInfo.Name='ENV_Action';

        this = this@rl.env.MATLABEnvironment(ObservationInfo, ActionInfo);

        updateActionInfo(this);
        end


        % Apply system dynamics and simulates the environment with the
        % given action for one step.
        function [Observation, Reward , IsDone, LoggedSignals] = step(this,Action)

            LoggedSignals = [];

            %Get action
            action = getForce(this,Action);

            % Unpack State eliments
            x_c1 = this.State(1,1);
            x_c2 = this.State(1,2);
            x_c3 = this.State(1,3);
            x_c4 = this.State(1,4);

            y_c1 = this.State(2,1);
            y_c2 = this.State(2,2);
            y_c3 = this.State(2,3);
            y_c4 = this.State(2,4);

            if action == -1

                x_c1 = x_c1 -1;
                x_c2 = x_c1;
                x_c3 = x_c3 -1;
                x_c4 = x_c3;

                y_c1 = y_c1 - 1;
                y_c2 = y_c2 - 1;
                y_c3 = y_c3 - 1;
                y_c4 = y_c4 - 1;

                this.act_num = -1;

            elseif action == 0

                y_c1 = y_c1 - 1;
                y_c2 = y_c2 - 1;
                y_c3 = y_c3 - 1;
                y_c4 = y_c4 - 1;

                if (x_c1 >=4 && x_c1 <= 10)
                    this.act_num = 1;
                else
                    this.act_num = -1;


                end

            elseif action == 1
                x_c1 = x_c1 +1;
                x_c2 = x_c1;
                x_c3 = x_c3 +1;
                x_c4 = x_c3;

                y_c1 = y_c1 - 1;
                y_c2 = y_c2 - 1;
                y_c3 = y_c3 - 1;
                y_c4 = y_c4 - 1;


                this.act_num = 1;


            end
            Observation = [x_c1, x_c2, x_c3, x_c4 ; y_c1, y_c2, y_c3, y_c4];
            % Update states
            this.State = Observation;

            % Check env

            IsDone =  this.State(2,1) == 0 ||  this.State(2,4) == 0; % If it is, IsDone = 1(true)
            this.IsDone = IsDone; % true

            % Get Reward
            Reward = getReward(this);

            notifyEnvUpdated(this);

        end

        % Reset environment to initial state and output initial observation
          % reset env to Initial Start State
        function InitialObservation = reset(this)
            x_c1 = randi([0 18]);
            x_c2 = x_c1;
            x_c3 = x_c1 + 2;
            x_c4 = x_c3;
            y_c1 = 18;
            y_c3 = 20;
            y_c2 = 20;
            y_c4 = 18;

            InitialObservation = [x_c1, x_c2, x_c3, x_c4 ; y_c1, y_c2, y_c3, y_c4];
            this.State = InitialObservation;

            % use notifyEnvUpdated to signal for update the visualization.
            notifyEnvUpdated(this);

        end

    end
    %% Optional Methods (set methods' attributes accordingly)
    methods

        % Discrete force 1 or 2
        function force = getForce(this,action)
            if ~ismember(action,this.ActionInfo.Elements)
                error(message('rl:env:CartPoleDiscreteInvalidAction',sprintf('%g',-this.MaxForce),sprintf('%g',this.MaxForce)));
            end
            force = action;           
        end

        %update the action info based on max force
        function updateActionInfo(this)
            this.ActionInfo.Elements = this.MaxForce * [-1 0 1];
        end




        function Reward = getReward(this)
            if this.State(2,1) == 0 && (this.State(1,1) >= 4 && this.State(1,1) <= 10)
                Reward = this.Reward;
            elseif this.State(2,1) == 0 && (this.State(1,1) < 4 || this.State(1,1) > 10)
                Reward = this.Penalty;
            elseif ~this.IsDone
                Reward = this.act_num;
                %this.Reward_A = this.Reward_A + this.act_num; % accumulate immediate reward
            end
        end
    end
    end





#### train code


    env = my_ENV ;
    %rlCreateEnvTemplate("my_ENV");

    rng(0)

    statePath = [
        imageInputLayer([2 4 1],'Normalization','none','Name','state')
        convolution2dLayer([1 2],3, 'Name','convolution1')
        fullyConnectedLayer(8,'Name','CriticStateFC1')
        reluLayer('Name','CriticRelu1')
        fullyConnectedLayer(8,'Name','CriticStateFC2')];

    actionPath = [
        imageInputLayer([1 1 1],'Normalization','none','Name','action')
        fullyConnectedLayer(8,'Name','CriticActionFC1')];
    commonPath = [
        additionLayer(2,'Name','add')
        reluLayer('Name','CriticCommonRelu')
        fullyConnectedLayer(1,'Name','output')];
    criticNetwork = layerGraph(statePath);
    criticNetwork = addLayers(criticNetwork, actionPath);
    criticNetwork = addLayers(criticNetwork, commonPath);    
    criticNetwork = connectLayers(criticNetwork,'CriticStateFC2','add/in1');
    criticNetwork = connectLayers(criticNetwork,'CriticActionFC1','add/in2');

    figure
    plot(criticNetwork)

    criticOpts = rlRepresentationOptions('LearnRate',0.01,'GradientThreshold',1);

    obsInfo = getObservationInfo(env);
    actInfo = getActionInfo(env);
    critic = rlQValueRepresentation(criticNetwork,obsInfo,actInfo,'Observation',{'state'},'Action',{'action'},criticOpts);

    agentOpts = rlDQNAgentOptions(...
        'UseDoubleDQN',false, ...    
        'TargetUpdateMethod',"periodic", ...
        'TargetUpdateFrequency',4, ...   
        'ExperienceBufferLength',30000, ...
        'DiscountFactor',0.99, ...
        'MiniBatchSize',256);


    agent = rlDQNAgent(critic,agentOpts);


    trainOpts = rlTrainingOptions(...
        'MaxEpisodes', 2000, ...
        'MaxStepsPerEpisode', 18, ...
        'Verbose', true, ...
        'Plots','training-progress',...
        'StopTrainingCriteria','AverageReward',...
        'StopTrainingValue',60);

    %plot(env)

    doTraining = true;
    if doTraining    
        % Train the agent.
        trainingStats = train(agent,env,trainOpts);
    else
        % Load pretrained agent for the example.
        load('MATLABCartpoleDQN.mat','agent');
    end


액션은 1과 0,-1이고 가장 최단거리로 효율적으로 움직이도록 보상을 계산한다.
한 step 움직일때마다 패널티가 붙는다.
