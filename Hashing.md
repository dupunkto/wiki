Hashing is a way to create a signature of some input.

Basically it works like this: you have some input (forexample the number 3) and it gets ran thru a big ass function that produces some weird output (`1121cfccd5913f0a63fec40a6ffd44ea64f9dc135c66634ba001d10bcf4302a2` in this case). You cannot get the original input from the output, but if you put the same input into the function again, it will produce the same output.

This is useful for a bunch of purposes:

- Storing passwords
- [[Hashmap]]

## Storing passwords

In the past, when you signed up for a website, your password was stored in their database. However, we quickly discovered that's a REALLY bad idea, for a few reasons:

- The owner of the site can see your password
- If the database gets stolen anyone can access your account

Combined with the fact that people reuse passwords often, I think you can see the problem.
The solution? Use a hash.

Instead of storing the password, you store its hash. When the user tries to log in, you compute the hash of the entered password, by running it thru the has algorithm, and if the output matches the stored hash in the database, the password was correct!

Because of this, sites don't literally have your password anymore. That's why they don't email you your password if you click the "Password forgotten" button.

