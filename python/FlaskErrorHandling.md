### Error Handling

- ## Custom Error Pages

  `@app.errorhandler(404)`

  `@app.errorhandler(500)`

  ```python
  @app.errorhandler(500)
  def internal_error(error):
      db.session.rollback()
      return render_template('500.html'), 500
  ```

  - `error handlers registered with Flask`

    `from app import routes, models, errors`

- ## Sending Errors by Email

  - The first step is to add the email server details to the configuration file:

    ```python
    # config admin email
    MAIL_SERVER = os.environ.get('MAIL_SERVER')
    MAIL_PORT = int(os.environ.get('MAIL_PORT') or 25)
    MAIL_USE_TLS = os.environ.get('MAIL_USE_TLS') is not None
    MAIL_USERNAME = os.environ.get('MAIL_USERNAME')
    MAIL_PASSWORD = os.environ.get('MAIL_PASSWORD')
    ADMINS = ['your-email@example.com']
    ```

  - > Flask uses Python's `logging` package to write its logs, and this package already has the ability to send logs by email. All I need to do to get emails sent out on errors is to add a [SMTPHandler](https://docs.python.org/3.6/library/logging.handlers.html#smtphandler)instance to the Flask logger object, which is `app.logger`:

    ```python
    # send error email
    if not app.debug:
        if app.config['MAIL_SERVER']:
            auth = None
            if app.config['MAIL_USERNAME'] or app.config['MAIL_PASSWORD']:
                auth = (app.config['MAIL_USERNAME'], app.config['MAIL_PASSWORD'])
            secure = None
            if app.config['MAIL_USE_TLS']:
                secure = ()
            mail_handler = SMTPHandler(
                mailhost=(app.config['MAIL_SERVER'], app.config['MAIL_PORT']),
                fromaddr='no-reply@' + app.config['MAIL_SERVER'],
                toaddrs=app.config['ADMINS'],
                subject='Microblog Failure',
                credentials=auth,
                secure=secure
            )
            mail_handler.setLevel(logging.ERROR)
            app.logger.addHandler(mail_handler)
    ```

