# Developing a Neural Network Regression Model

## AIM
To develop a neural network regression model for the given dataset.

## THEORY
Explain the problem statement

## Neural Network Model
Include the neural network model diagram.

## DESIGN STEPS
### STEP 1: 

Create your dataset in a Google sheet with one numeric input and one numeric output.

### STEP 2: 

Split the dataset into training and testing

### STEP 3: 

Create MinMaxScalar objects ,fit the model and transform the data.

### STEP 4: 

Build the Neural Network Model and compile the model.

### STEP 5: 

Train the model with the training data.

### STEP 6: 

Plot the performance plot

### STEP 7: 

Evaluate the model with the testing data.

### STEP 8: 

Use the trained model to predict  for a new input value .

## PROGRAM

### Name:

### Register Number:

```python
class NeuralNet(nn.Module):
    def __init__(self):
        super().__init__()
        #Include your code here


import torch
import torch.nn as nn
import torch.optim as optim
import pandas as pd
from sklearn.model_selection import train_test_split
from sklearn.preprocessing import MinMaxScaler
     

dataset1 = pd.read_csv('           ')
X = dataset1[['Input']].values
y = dataset1[['Output']].values
     

X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.33, random_state=33)
     

scaler = MinMaxScaler()
X_train = scaler.fit_transform(X_train)
X_test = scaler.transform(X_test)
     

X_train_tensor = torch.tensor(X_train, dtype=torch.float32)
y_train_tensor = torch.tensor(y_train, dtype=torch.float32).view(-1, 1)
X_test_tensor = torch.tensor(X_test, dtype=torch.float32)
y_test_tensor = torch.tensor(y_test, dtype=torch.float32).view(-1, 1)
     

# Name:
# Register Number:
class NeuralNet(nn.Module):
  def __init__(self):
        super().__init__()
        # Include your code here





     

# Initialize the Model, Loss Function, and Optimizer
# Write your code here
     

# Name:
# Register Number:
def train_model(ai_brain, X_train, y_train, criterion, optimizer, epochs=2000):
    # Write your code here




        ai_brain.history['loss'].append(loss.item())
        if epoch % 200 == 0:
            print(f'Epoch [{epoch}/{epochs}], Loss: {loss.item():.6f}')

     

train_model(ai_brain, X_train_tensor, y_train_tensor, criterion, optimizer)

     

with torch.no_grad():
    test_loss = criterion(ai_brain(X_test_tensor), y_test_tensor)
    print(f'Test Loss: {test_loss.item():.6f}')

     

loss_df = pd.DataFrame(ai_brain.history)
     

import matplotlib.pyplot as plt
loss_df.plot()
plt.xlabel("Epochs")
plt.ylabel("Loss")
plt.title("Loss during Training")
plt.show()
     

X_n1_1 = torch.tensor([[9]], dtype=torch.float32)
prediction = ai_brain(torch.tensor(scaler.transform(X_n1_1), dtype=torch.float32)).item()
print(f'Prediction: {prediction}')
     


# Initialize the Model, Loss Function, and Optimizer



def train_model(ai_brain, X_train, y_train, criterion, optimizer, epochs=2000):
    #Include your code here

```

### Dataset Information
<img width="390" height="568" alt="image" src="https://github.com/user-attachments/assets/74e4a105-20f1-4782-9a5d-e81a38e161c1" />

### OUTPUT
<img width="548" height="78" alt="image" src="https://github.com/user-attachments/assets/dba18d3a-d165-42c9-bcf9-14cbeb5588cf" />

### Training Loss Vs Iteration Plot
<img width="1310" height="1028" alt="image" src="https://github.com/user-attachments/assets/3c1a71aa-42cc-4ddb-8c86-381d086e27a0" />

### New Sample Data Prediction
<img width="474" height="69" alt="image" src="https://github.com/user-attachments/assets/3ed77243-873d-4143-91dd-c07bb4a22867" />

## RESULT
Thus, a neural network regression model was successfully developed and trained using PyTorch.
