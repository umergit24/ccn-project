Computer and Communication Networks 
INSTRUCTOR: Dr. Hassaan Khaliq Qureshi 
COURSE CODE: EE357 
SEMESTER: 6th 

PROJECT BY: Umer Abdullah Khan
CMS: 375870


# **Create a containerized weather application available as a web front end using Minikube.**
# 1. Create a python application that uses OpenWeatherAPI to get the weather data.



```ad-note
Codes for python and html in Appendix ([[#7. Appendix]]) 
```


.
.
.
.




![[Pasted image 20240413141925.png]]
(weather forecasted for next 5 days with 3 hour step)
![[Pasted image 20240413142017.png]]
![[Pasted image 20240413142039.png]]
.
.
.
![[Pasted image 20240413142106.png]]







# 2. Using Docker to containerize this application.

#### Dockerfile

```Dockerfile
FROM python:3.10.11-slim

ADD weather.py .

WORKDIR /ccnproject

COPY requirements.txt requirements.txt

RUN pip install -r requirements.txt

COPY . .

ENTRYPOINT [ "python3"]

CMD [ "./weather.py" ]
```

#### Building Docker Image

```linux
docker build -t ukahn2003/pythonapp_image .
```

![[Pasted image 20240411231511.png]]

#### Pushing image to Docker Hub Registry

```linux
docker push ukhan2003/pythonapp_image
```

![[Pasted image 20240411231926.png]]

![[Pasted image 20240413131513.png]]
#### Running the container
```linux
docker run -p 3000:3000 ukhan2003/pythonapp_image
```


![[Pasted image 20240411233815.png]]

#### Running the containerized python app 
( it works 👍)
 ![[Pasted image 20240411233453.png]]

# 3. Creating a pod in Minikube using our image

Check that Minikube is properly installed, by running the _minikube version_ command:
![[Pasted image 20240411185214.png]]
Now we can start the cluster by running the _minikube start_ command.
View the cluster details by running ;
```linux
kubectl cluster-info 
```

![[Pasted image 20240411235410.png]]

> [!NOTE]
> Cluster IP = 192.168.49.2

To view the nodes in the cluster, run;

```linux
kubectl get nodes
```

![[Pasted image 20240411235444.png]]

Let’s deploy our first app on Kubernetes.

```linux
nano deployment.yaml
```

This will create a YAML file for us where we can add the definitions for deployments as well as service which will be needed in the next task. The deployment and service definitions are separated using "---".

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: pythonapp-service
spec:
  selector:
    matchLabels:
      app: pythonapp
  replicas: 1
  template:
    metadata:
      labels:
        app: pythonapp
    spec:
      containers:
        - name: flask-test-app
          image: docker.io/ukhan2003/pythonapp_image
          imagePullPolicy: IfNotPresent
          ports:
            - containerPort: 3000
---

apiVersion: v1
kind: Service
metadata:
  name: pythonapp-deploy
spec:
  selector:
    app: pythonapp
  ports:
    - protocol: "TCP"
      port: 6000
      targetPort: 3000
  type: LoadBalancer

```

File details and explanations:
.
.
.
.

.

```kubectl
kubectl apply -f deployment.yaml
```

![[Pasted image 20240412194531.png]]

```linux
kubectl get all
```

![[Pasted image 20240412194751.png]]




# 4. Expose this pod on a port (using service)

We can use the command below combined with the name of our service

```yaml
minikube service pythonapp-deploy
```

This automatically open the application on the browser. It should display as shown below;

![[Pasted image 20240412195038.png]]

(Works 👍)

![[Pasted image 20240412195409.png]]



# 5 .Observe and interact with this application on your host machine

To make out host access the application, we can use port-forwarding.
.
> [!NOTE] INFO
> HOST : WINDOWS 11 OS
> VM : UBUNTU 22.04


```linux
kubectl port-forward --address localhost,192.168.18.159 deployment/pythonapp-service 8080:3000
```
![[Pasted image 20240413023815.png]]

![[Pasted image 20240413023120.png]]
![[Pasted image 20240413023210.png]]

# 6. Deploy the Minikube dashboard to observe statistics related to your cluster.

Viewing the Minikube Dashboard

![[Pasted image 20240412195739.png]]![[Pasted image 20240412195807.png]]
![[Pasted image 20240413023658.png]]



# 7. Appendix
## A: Python Code ([[#1. Create a python application that uses OpenWeatherAPI to get the weather data.]])
"weather.py"
```python
from flask import Flask, request, render_template
import requests
import datetime as dt

app = Flask(__name__)

API_KEY = '007d68128ef4ec788f611c7c0cc7f68a'

@app.route('/', methods=["POST", "GET"])

def search_city():
    if request.method == "POST":
        city = request.form.get("city")
        if city == "" or len(city) <= 1:
            error_message = "City is required."
            return render_template("weather.html", error_message=error_message)
        units = 'Metric'
        url = f'http://api.openweathermap.org/data/2.5/forecast?q={city}&APPID={API_KEY}&units={units}'
        response = requests.get(url).json()
        if response["cod"] != "200":
            error_message = "City not found."
            return render_template("weather.html", error_message=error_message)
        forecast_data = response["list"]
        forecast = []
        for item in forecast_data:
            forecast_date = dt.datetime.fromtimestamp(item["dt"]).strftime('%d-%m-%Y')
            forecast_temp = item["main"]["temp"]
            forecast_desc = item["weather"][0]["description"]
            forecast_icon = item["weather"][0]["icon"]
            forecast.append({"date": forecast_date, "temp": forecast_temp, "desc": forecast_desc, "icon": forecast_icon})
        return render_template("weather.html", city=city, forecast=forecast)
    return render_template("weather.html")
if __name__ == '__main__':
    app.run(host="0.0.0.0", port=3000, debug=True)
```

## B: HTML code ([[#1. Create a python application that uses OpenWeatherAPI to get the weather data.]])

```html
<!DOCTYPE html>

<html>
<head>
  <title>Weather App</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      background-color: #f2f2f2;
      margin: 0;
      padding: 0;
    }
  
    .header {
      background-color: #007bff;
      color: #fff;
      padding: 20px;
      text-align: center;
    }
  
    .container {
      max-width: 800px;
      margin: 0 auto;
      padding: 20px;
    }


    h1 {
      margin-top: 0;
      font-size: 32px;
    }

    p {
      margin: 0;
      font-size: 16px;
    }

    form {
      margin-bottom: 20px;
      text-align: center;
    }

    input[type="text"] {
      padding: 10px;
      font-size: 16px;
      width: 200px;
    }

    button {
      padding: 10px 20px;
      font-size: 16px;
      background-color: #007bff;
      color: #fff;
      border: none;
      cursor: pointer;
    }
  
    table {
      width: 100%;
      border-collapse: collapse;
      margin-top: 20px;
    }

    th, td {
      padding: 12px;
      text-align: left;
      border-bottom: 1px solid #ddd;
    }

  
    .weather-icon {
      width: 30px;
      height: 30px;
    }

  </style>
</head>
<body>
  <div class="header">
  
    <h1>Weather App</h1>

    <p>by Umer Abdullah Khan</p>

  </div>

  <div class="container">

    <form method="POST">

      <input name="city" type="text" placeholder="City Name">

      <button>Get Weather</button>

    </form>

  

    {% if error_message %}

      <p>{{ error_message }}</p>

    {% endif %}

  

    {% if forecast %}

      <h2>{{ city }}</h2>

      <table>

        <tr>

          <th>Date</th>

          <th>Temperature</th>

          <th>Description</th>

          <th>Icon</th>

        </tr>

        {% for entry in forecast %}

          <tr>

            <td>{{ entry.date }}</td>

            <td>{{ entry.temp }}°C</td>

            <td>{{ entry.desc }}</td>

            <td><img class="weather-icon" src="http://openweathermap.org/img/w/{{ entry.icon }}.png" alt="Weather Icon"></td>

          </tr>

        {% endfor %}

      </table>

    {% endif %}

  </div>

</body>

</html>
```


