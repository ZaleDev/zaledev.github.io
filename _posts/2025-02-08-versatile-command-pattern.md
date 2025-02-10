---
layout: post
title: Versatile Command Pattern
date: 2025-02-08 17:59 +0000
description: Implementation of the Command pattern in Unreal Engine C++ and Blueprint.
categories: [Unreal, Design Patterns]
tag: [unreal, design patterns, command]
toc: false
---

This post demonstrates a versatile implementation of the Command pattern in Unreal Engine. The approach leverages both C++ and Blueprints, encapsulating commands as objects for flexible and decoupled design.

### Base Class

We begin with a base class (named "BlueprintableObject") that provides a valid GetWorld() implementation. This is essential for operations such as actor spawning, accessing gameplay classes, and timer management.

```cpp
// BlueprintableObject.h

UCLASS(Abstract, Blueprintable, BlueprintType)
class MYGAME_API UBlueprintableObject : public UObject
{
	GENERATED_BODY()

public:
	virtual UWorld* GetWorld() const override;
	
};
```

```cpp
// BlueprintableObject.cpp

UWorld* UBlueprintableObject::GetWorld() const
{
#if WITH_EDITOR
	if (IsTemplate()) 
	{
		return nullptr;
	} 
#endif

	return Super::GetWorld();
}
```

### Command Actions and Validation Checks

The implementation is divided into two parts: one for command execution and another for condition validation.

#### Action Commands

The UActionObject class encapsulates an action command. Marked as a BlueprintNativeEvent, it allows for modifications in both C++ and Blueprints.

```cpp
// ActionObject.h

UCLASS(Abstract, Blueprintable, BlueprintType)
class MYGAME_API UActionObject : public UBlueprintableObject
{
	GENERATED_BODY()

public:
	UFUNCTION(BlueprintNativeEvent)
	void Execute();
	virtual void Execute_Implementation() {};
	
};
```

#### Check Commands

The UCheckObject class implements a validation command that returns a boolean outcome, making it useful for condition checking in gameplay logic.

```cpp
// CheckObject.h

UCLASS(Abstract, Blueprintable, BlueprintType)
class MYGAME_API UCheckObject : public UBlueprintableObject
{
	GENERATED_BODY()

public:
	UFUNCTION(BlueprintNativeEvent)
	bool Check();
	virtual bool Check_Implementation() { return false; };
	
};
```

### Practical Example - Dialogue System

In a dialogue scenario, a player may face multiple dialogue choices, yet some options should only appear if certain conditions are satisfied. By utilizing the command pattern, we can encapsulate these conditions in dedicated check objects. For example, consider a check object named Check_Relationship that verifies if the player has reached a minimum relationship level with a character.

Each check object can be parameterized — Check_Relationship may include:
- An integer parameter "min_relationship" specifying the required minimum relationship value.
- A UGameplayTag parameter "character" to reference the character in question.

During dialogue processing:
- Gather an array of check objects associated with each dialogue choice.
- For each option, iterate through its checks by calling the Check() method.
- Only display the dialogue option if all its associated checks return true.

This approach simplifies the routing logic in complex dialogue systems, ensuring that each dialogue option dynamically reflects the game state.

### Weaknesses and Limitations

- Check objects have very limited context awareness. When a check is executed, it lacks information about the context (in the dialogue system example above, a check wouldn't be able to know which dialogue is calling it, or which character is talking. It would need to work only with global data and its own parameters to avoid additional coupling).
- The pattern assumes that all necessary parameters are provided externally, which may lead to rigid implementations if contextual data changes.
- Debugging can be more challenging since the logic is decoupled into multiple objects, possibly obscuring the overall flow.
- Overuse of this pattern might lead to an increased number of small, interdependent objects and can complicate maintenance.
