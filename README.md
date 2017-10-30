REST API Sample
===

**Bundles Required For REST API**

For making REST API with Symfony we must need FOSRestBundle but along with it we also need to install 2 more bundles:

- JMSSerializerBundle
- NelmioCorsBundle

FOSRestBundle is the backbone of REST API in Symfony. It accepts the client request and returns the appropriate response accordingly. JMSSerializerBundle helps in serialization of data according to your requested format namely json,xml or yaml. NelmiCorsBundle allows different domains to call our API.

**Bundles Registration**

After successfully installing the bundle, you need to register them in AppKernel.php file located in app folder. Open the file and add these lines in bundles array.
```ruby
new FOS\RestBundle\FOSRestBundle(),
new JMS\SerializerBundle\JMSSerializerBundle(),
new Nelmio\CorsBundle\NelmioCorsBundle(),
```
**Configuration in config.yml**

As I’ve said before, we need to enable our API to call different domains for outputting headers. This action needs a little configuration in config.yml file. Open it and add the following code in it:

```ruby
# Nelmio CORS Configuration
nelmio_cors:
    defaults:
        allow_credentials: false
        allow_origin: ['*']
        allow_headers: ['*']
        allow_methods: ['GET', 'POST', 'PUT', 'DELETE', 'OPTIONS']
        max_age: 3600
        hosts: []
        origin_regex: false
 
# FOSRest Configuration
fos_rest:
    body_listener: true
    format_listener:
        rules:
            - { path: '^/', priorities: ['json'], fallback_format: json, prefer_extension: false }
    param_fetcher_listener: true
    view:
        view_response_listener: 'force'
        formats:
            json: true
```

**Create a User Entity in Symfony**

We need some data to create, update, and delete. For this, we are going to make an entity User. To simplify things we just add only name and role of user to make different entities. 

Now our database is ready. You can insert some data in it by running insert query in your db
```ruby
INSERT INTO user (name, role) VALUES
 
('tommy', 'community manager'),
 
('sandy', 'digital content producer'),
 
( 'michael', 'php community manager');
```

**Getting records from database**

Create UserController.php file and add the following namespace and dependency file URLS at the top

```ruby
namespace AppBundle\Controller;
 
use Sensio\Bundle\FrameworkExtraBundle\Configuration\Route;
use Symfony\Bundle\FrameworkBundle\Controller\Controller;
use FOS\RestBundle\Controller\Annotations as Rest;
use FOS\RestBundle\Controller\FOSRestController;
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\HttpFoundation\Response;
use FOS\RestBundle\View\View;
use AppBundle\Entity\User;
```
After this, make first controller class UserController which extend FOSRestController. All the methods and request routes will be made in this class.

```ruby
class UserController extends FOSRestController
{
}
```
Make a first Get route and its method to retrieve user data from database. if there is no record exist it will show the response message in view. Copy the code below into the UserController class:

```ruby
**
* @Rest\Get("/user")
*/
public function getAction()
{
    $restresult = $this->getDoctrine()->getRepository('AppBundle:User')->findAll();
    if ($restresult === null) {
        return new View("there are no users exist", Response::HTTP_NOT_FOUND);
    }
    return $restresult;
}
```
***We are going to use postman to test our API***<br/>

Lets check if the data is retrieved or not in our API call in postman. You need to find your application URL in Application. Copy the URL in postman with /user suffix.


If you want to retrieve only one record then pass the user ID in idAction() and find() method. you can make a new route and method like this
```ruby
/**
 * @Rest\Get("/user/{id}")
 */
 public function idAction($id)
 {
   $singleresult = $this->getDoctrine()->getRepository('AppBundle:User')->find($id);
   if ($singleresult === null) {
   return new View("user not found", Response::HTTP_NOT_FOUND);
   }
 return $singleresult;
 }
```
**Posting records in database**

To post new record in database add the following code in UserController class.
You can check by running the GET call to fetch all users whether the user added or not.
```ruby
 /**
 * @Rest\Post("/user/")
 */
 public function postAction(Request $request)
 {
   $data = new User;
   $name = $request->get('name');
   $role = $request->get('role');
   if(empty($name) || empty($role))
   {
       return new View("NULL VALUES ARE NOT ALLOWED", Response::HTTP_NOT_ACCEPTABLE); 
   } 
  $data->setName($name);
  $data->setRole($role);
  $em = $this->getDoctrine()->getManager();
  $em->persist($data);
  $em->flush();
  return new View("User Added Successfully", Response::HTTP_OK);
 }
```
**Updating record in database**

To update record, add PUT call in route and write the code provided below. Basically it will take the ID of the user to retrieve the name and role of user, after which you can edit the fields and update it by PUT header in postman.
```ruby
 /**
 * @Rest\Put("/user/{id}")
 */
 public function updateAction($id,Request $request)
 { 
 $data = new User;
 $name = $request->get('name');
 $role = $request->get('role');
 $sn = $this->getDoctrine()->getManager();
 $user = $this->getDoctrine()->getRepository('AppBundle:User')->find($id);
if (empty($user)) {
   return new View("user not found", Response::HTTP_NOT_FOUND);
 } 
elseif(!empty($name) && !empty($role)){
   $user->setName($name);
   $user->setRole($role);
   $sn->flush();
   return new View("User Updated Successfully", Response::HTTP_OK);
 }
elseif(empty($name) && !empty($role)){
   $user->setRole($role);
   $sn->flush();
   return new View("role Updated Successfully", Response::HTTP_OK);
}
elseif(!empty($name) && empty($role)){
 $user->setName($name);
 $sn->flush();
 return new View("User Name Updated Successfully", Response::HTTP_OK); 
}
else return new View("User name or role cannot be empty", Response::HTTP_NOT_ACCEPTABLE); 
 }
```
Now move to postman enter the user id in URL and change the header to PUT. Add the update records in Params and send the request.

**Deleting record in database**

To delete records via API call, you need to pass the ID of user in URL. Add the below route and method in class:
```ruby
 /**
 * @Rest\Delete("/user/{id}")
 */
 public function deleteAction($id)
 {
  $data = new User;
  $sn = $this->getDoctrine()->getManager();
  $user = $this->getDoctrine()->getRepository('AppBundle:User')->find($id);
if (empty($user)) {
  return new View("user not found", Response::HTTP_NOT_FOUND);
 }
 else {
  $sn->remove($user);
  $sn->flush();
 }
  return new View("deleted successfully", Response::HTTP_OK);
 }
```
Enjoy