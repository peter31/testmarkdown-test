#### Injected services

So, the default form handler service has injection of form factory, twig templating and session services, and in addition - router, entity manager, user manager and finally - the service container itself.

object          | description
----------------|---------------------------------------------------------------------
**formFactory** | Generates form presentation of the form type prepared in the handler
**router**      | For redirection on success/failure
**templating**  | Renders the form presentation
**em**          | Saves the model
**userManager** | Mostly for interacting with the current user 's entities
**session**     | Stores flash messages and probably saves model data
**container**   | To retrieve more services not covered by constructor

