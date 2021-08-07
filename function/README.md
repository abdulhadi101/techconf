## This folder will contains the Azure function code.

## Note:

- Before deploying, be sure to update your requirements.txt file by running `pip freeze > requirements.txt`
- Known issue, the python package `psycopg2` does not work directly in Azure; install `psycopg2-binary` instead to use the `psycopg2` library in Azure

The skelton of the `__init__.py` file will consist of the following logic:

```
import logging
import azure.functions as func
import psycopg2
import os
from datetime import datetime
from sendgrid import SendGridAPIClient
from sendgrid.helpers.mail import Mail

def main(msg: func.ServiceBusMessage):

    notification_id = int(msg.get_body().decode('utf-8'))
    logging.info('Python ServiceBus queue trigger processed message: %s',notification_id)

    # TODO: Get connection to database
    connection = psycopg2.connect(user = "udacity@uda-postgresql",
                                  password = "MyPostgresPass",
                                  host = "uda-postgresql.postgres.database.azure.com",
                                  port = "5432",
                                  database = "techconfdb")

    try:
        # TODO: Get notification message and subject from database using the notification_id
        notification_query = '''SELECT subject, message 
                                FROM Notification
                                WHERE id = %s;'''

        cursor.execute(notification_query, (notification_id,))
        notification = cursor.fetchone()

        subject = notification[0]
        message = notification[1]
        
        # TODO: Get attendees email and name
        attendees_query = 'SELECT first_name, email FROM Attendee;'
        cursor.execute(attendees_query)
        attendees = cursor.fetchall()

        # TODO: Loop through each attendee and send an email with a personalized subject
        for attendee in attendees:
          message = Mail(
              from_email='from_email@example.com',
              to_emails=attendee[0],
              subject='{}: {}'.format(attendee[1], subject),
              html_content=messagePlain)
          try:
              sg = SendGridAPIClient(os.environ.get('SENDGRID_API_KEY'))
              response = sg.send(message)
              print(response.status_code)
              print(response.body)
              print(response.headers)
          except Exception as e:
              print(str(e))

        # TODO: Update the notification table by setting the completed date and updating the status with the total number of attendees notified

    except (Exception, psycopg2.DatabaseError) as error:
        logging.error(error)
    finally:
        # TODO: Close connection
```
