# SingleDelegates

# Definitions 

- Delegates / Invokers / Broadcasters: functions that SEND (broadcast) an event.
- Callbacks / Listeners: functions that RECEIVE the event.

## Delegates
  - Delegates are used in event-driven programming
  - They are functions that call other functions (callback functions or listeners) when an event occurs.
  - They are bound to their callback functions storing references to these functions and trigger their action when an event occurs.
  - The list of callback functions or member functions that are bound to the delegates is called invocation list 

# Definition of Single Cast Delegates
  - Single cast delegates can be bound to only one function. 

# Types of Single Delegates:
  - DECLARE_DELEGATE – single-cast delegate.
  - DECLARE_DELEGATE_<Num>Params – single-cast delegate with Num parameter.
  - DECLARE_DELEGATE_RetVal_<Num>Params – single-cast delegate with a return value and Num parameters.
  - DECLARE_DYNAMIC_DELEGATE – dynamic single-cast delegate.
    - Whereas with other types of delegates their invocation list is set at compile time, Dynamic single cast delegate can dynamically change their invocation list at run time.
    - This allows them to add or remove callback functions according to user input, game state or other runtime conditions. 
    
# 0- Create Classes
  - Create a Sender C++ class of AActor type that will send some information using the delegate
  - Create a Receiver C++ class of AActor type that will receive the information and use this to execute the callback function
  - Create a blueprint based on the Sender C++ class
  - Create a blueprint based on the Receiver C++ class
  - Place both blueprints in the world
  
  ### DELEGATE: DECLARE (TEMPLATE) > DEFINE (EXECUTE) > CALLBACK: DECLARE > DEFINE > FIND > BIND > UNBIND
  
# 1- DECLARE THE DELEGATE FUNCTION

  - In the Sender class header file, Declare the type of delegate that I am going to use.
    - MACRO OF DELEGATES DECLARATION<num of params>(name of this delegate, type of the param this class will send to the receiver)
  - Declare a Send() function to contain from which the delegate will be executed 
  - Declare a delegate object that will be used to bound my delegate function to its callback function 
  
  ```cpp
DECLARE_DELEGATE_OneParam(FSenderDelegate, FString); 

UCLASS()
class MYPROJECT_API ASender : public AActor
{
	GENERATED_BODY()
	
public:	
	// Sets default values for this actor's properties
	ASender();

	//Declare a delegate function
	void Send();

protected:
	// Called when the game starts or when spawned
	virtual void BeginPlay() override;

public:	
	// Called every frame
	virtual void Tick(float DeltaTime) override;

	//1.2- Declare a delegate object that will be used to bound my delegate function to its callback function
	FSenderDelegate MySenderDelegate;

};
  ```
  
  # 2- DEFINE THE DELEGATE FUNCTION 
  
  - In the implementation file, define the sender function, inside of it execute the delegate if it is really bound to a callback function and pass the parameter that will be sent to the callback function - respecting the parameter type defined in the delegate function declaration.
  
  ```cpp
  void ASender::BeginPlay()
{
	Super::BeginPlay();
}

//DELEGATE
	//2- Define the delegate funtion
void ASender::Send()
{
	//Use the delegate object to pass the value that will be sent to the Callback function - if this delegate object is bound to any function
	UE_LOG(LogTemp, Warning, TEXT("Delegate function SEND is executing, calling the Callback function and passing a parameter"));
	MySenderDelegate.ExecuteIfBound("This is a parameter from Sender.");
}
  ```
  
  # 3- DECLARE THE CALLBACK FUNCTION
  
  - In the Receiver class header file, Declare an object from the Sender class to access its delegate function and bind it to the receiver callback function
  - Declare a Receive() function that will be our callback function and will be called when the delegate executes and expose it with UFUNCTION() - its parameter type should be the same as that of the delegate function that will call it.
  - Declare an EndPlay funtion and override its parent
    - Will be used to unbind he the Callback function from the Delegate function if Sender or Receiver die or move out of the game
	
```cpp
private:
	//Declare a pointer object to my sender class
	ASender* MySender;

public:	
	// Sets default values for this actor's properties
	AReceiver();

	//3- Declare the Callback function
	UFUNCTION()
	void Receive(FString Message);

	//Declare an EndPlay funtion and override its parent
		//Will be used to unbind he the Callback function from the Delegate function if Sender or Receiver die or move out of the game
	UFUNCTION()
	virtual void EndPlay(const EEndPlayReason::Type RemovedFromWorld) override;
```

  # 4- DEFINE THE CALLBACK FUNCTION
  - Define the callback function to Receive and print message
	
```cpp
void AReceiver::Receive(FString Message)
{
	UE_LOG(LogTemp, Warning, TEXT("Executing the Callback function RECEIVE and receiving the parameter from the Delegate"));
	UE_LOG(LogTemp, Warning, TEXT("This is my message : %s "), *Message);
}
```

  # 5- FIND THE DELEGATE
  - In Unreal Engine, Project Settings, create a tag for objects from the "Sender" class
  - Inside the sender actor blueprint, create and select a sender tag
	
  - In the Receiver class implementation file, Find the all actors with tag "Sender" and place them into an array of actors.
  - Get the first sender actor of the array, cast it type ASender, and save into the sender object

```cpp
void AReceiver::BeginPlay()
{
	Super::BeginPlay();

	//5- FIND DELEGATE:
		//Find the all actors with tag "Sender" and place them into an array of actors
		//Get the first sender actor of the array, cast it type ASender, and save into the sender object
		//In Unreal, create a "Sender" tag in project settings and include it in the Sender actor BP

	TArray<AActor*> TaggedActors;
	UGameplayStatics::GetAllActorsWithTag(GetWorld(), "Sender", TaggedActors);
	MySender = Cast<ASender>(TaggedActors[0]);
}
```

  # 6- BIND THE DELEGATE TO THE CALLBACK FUNCTION

  - In the Receiver class BeginPlay(), Access the delegate object from the Sender class and use it to Bind this object "Sender" to the callback funtion "Receive"
	
```cpp
void AReceiver::BeginPlay()
{
	Super::BeginPlay();
	
	//6- BIND
		//Access the delegate object from the Sender class and use it to Bind this object "Sender" to the callback funtion "Receive"
	MySender->MySenderDelegate.BindUObject(this, &AReceiver::Receive);
}
``` 
	
  # 7- UNBIND THE DELEGATE FROM THE CALLBACK FUNCTION
	
  - On EndPlay() Access the delegate object from the sender class and use the Unbind function to unbind it from its callback function

```cpp
void AReceiver::EndPlay(const EEndPlayReason::Type)
{
	MySender->MySenderDelegate.Unbind();
}
``` 
	
	
	
	
	
	
	
