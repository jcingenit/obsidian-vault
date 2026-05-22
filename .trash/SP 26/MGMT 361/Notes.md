## Process Analysis
### Process Flow Diagram

Process View
	Process - Efficient and effective management of process that create goods / provide services
	Inputs -> Outputs by Resources
	Inputs Vs Resources?
		Resources not incorporated into outputs
	Processes can involve both goods and services
	Processes can have multiple inputs / multiple outputs
Flow Unit
	What is tracked through the process, generally defines process output of interest.
	Work-in-process are inventories kept in a process
Elements of a Process
	Activities - Squares
		Carried out by Resources
		Value-adding (necessary)
		Have Capacity
	Flow - Arrow
		Indicate flow of the unit
	Inventory / Buffers - Triangle
		Do not have capacity but may have limited space
Resource Capacity
	Capacity of a resource = 1/Activity Time
		Activity Time (processing time)
	Bottleneck
		Stage with the SMALLEST capacity, LONGEST processing time
Process Capacity
	How much the process can product in a given time
		Determined by bottleneck capacity
		Min {Capacity of stage 1 .. K} = Bottleneck Resource Capacity 
	Bottleneck
		Stage with the SMALLEST capacity, LONGEST processing time
Designed Cycle Time
	Process DCT
		1 / Process Capacity
	Stage DCT
		1 / Stage Capacity
## Process Analysis
### Capacity Analysis

Important Concepts and Formulas
	Capacity of Resource = 1 / Activity Time
	Bottleneck = Stage with the SMALLEST capacity
	Process Capacity = Bottleneck Stage Capacity
	Direct Labor Costs = Total Wages / Process Capacity
	Actual Cycle Time = 1 / Flow Rate
		Flow Rate = Min {ProcessCapacity, DemandRate}

## Process Analysis
### Run Time Parameters

Line Balancing
	Reallocate tasks so Activity time for each resource is more or less balanced
Design Parameters
	Process Capacity
	Designed Cycle Time

Supply-Constrained Process
	Process Capacity < Demand Rate
	Flow Rate = Process Capacity
Demand-Constrained Process
	Process Capacity > Demand Rate
	Flow Rate = Demand Rate

Actual Cycle Time
	Supply-Constrained
		Process Capacity < Demand Rate
			Actual CT = Designed CT
	Demand-Constrained
		Process Capacity > Demand Rate
			Actual CT = Average Inter-Arrival Time

Utilization
	Stage Utilization
		= Flow Rate / Stage Capacity
	Process Utilization
		= Flow Rate / Process Capacity

Design Parameters
	Indicate what a process CAN achieve
Run-Time Parameters
	Indicate current process performance under a specific demand rate or customer arrival schedule

## Process Analysis
### Gantt Chart

![[ProcessMeasuresSummary.png]]

![[GanttChart.png]]

![[GanttUtilization.png]]

## Gantt Chart
### WIP, Flow Time

Flow Time
	Average Time to go through Entire Process
	Average Throughput Time
		May Contain Waiting Time (Time spent in buffer)

Cycle Time
	Time between consecutive units
		Every 30 seconds, a customer leaves X
		Every 1.5 min, a new car drives off the assembly line
Flow Time
	Time between Entry and Departure of the same unit
		Customer need 3 minutes on average to buy a sub
		90 Minutes to assemble a car

![[GanttFlowTime.png]]

LITTLE'S Formula
	Flow Rate * Flow Time = WIP
		Must be Demand-Constrained

![[LittlesSummary.png]]

Inventory Turns
	= 1 / Avg Time to Sell Inventory
	= 1 / Avg Time product Kept in Inventory

![[InventoryTurns.png]]

![[LittlesSummary2.png]]

### Batching and Setup Time

Batch
	= Collection of Flow Units that are processed during one production (service) cycle
	Important Features
		Multiple Units in one batch
			Delivered same time or continuously
		Starting a new batch needs a new setup
			Required by some resource (equipment, worker, etc)
		Setup takes time...

Batch Size & Capacity
	Capacity Given Batch Size
		= Batch Size / Setup Time + Batch Size * Time per unit
	Capacity Increases with Batch Size and Decreases with setup time
	Largest possible capacity happens when setup time is eleminated.
		Capacity = 1 / Time per Unit
