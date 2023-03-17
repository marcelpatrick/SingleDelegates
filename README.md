# SingleDelegates

# Definition of Delegates
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
  - Create a Sender class that will send some information using the delegate
  - Create a Receiver class that will receive the information and use this to execute the callback function
  
# 1- DECLARE THE DELEGATE FUNCTION

  - In the Sender class header file, Declare the type of delegate that I am going to use.
    - MACRO OF DELEGATES DECLARATION<num of params>(name of this delegate, type of the param this class will send to the receiver)
  - Declare a Send() function to contain and execute the delegate
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
  
  - In the implementation file, define the sender function from which the delegate will be executed and send its parameter to the callback function - respecting the parameter type defined in the delegate function declaration.
  - Call the sender function from BeginPlay()
  
  ```cpp
  void ASender::BeginPlay()
{
	Super::BeginPlay();

	Send();	
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
    
  
