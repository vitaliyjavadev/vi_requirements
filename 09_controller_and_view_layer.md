# controller_and_view_layer

## controller

Контроллеры это связующее звено между вашим приложением и реальным миром. К
контроллерам предъявляются следующие требования:

1. **Расположение контроллеров.** Контроллеры должны располагаться в пакете
   `{artifact_id}.<project>.controller`;
2. **Наименование контроллеров.** У контроллеров должен быть суффикс
   `Controller`, например, `PostController`, а не `PostControl`;
3. **Обработка ошибок.** Если при обработке происходит ошибка, то контроллер
   должен сообщать об этом пользователю. Но выброс исключений должен быть на
   уровне сервисов!

   Неправильный код:

   ```java
   // сервис
   public Task save(Task task) {
        return taskRepository.save(task);
   }
   // ...
   // контроллер
   public ModelAndView saveTask(@ModelAttribute("task") Task task) {
        Optional<User> userOptional = userService.findById(task.getUserId());
        if (userOptional.isEmpty()) {
            return new ModelAndView("error/404", "errorMessage", new ExceptionMessage("User with this id is not found"));
        }
        String title = task.getTitle();
        if (title == null) {
            return new ModelAndView("error/400", "errorMessage", new ExceptionMessage("Task title must not be empty"));
        }
        taskService.save(task);
        return new ModelAndView("task/list");
   }
   ```

   Этот код нарушает SRP в классе TaskController, потому что этот класс
   занимается и валидацией и обработкой веб-запросов.
   К тому же при смене контроллера, например, на контроллер очереди сообщений
   класс TaskService становится непригодным, потому что
   он не содержит логику, которую должен содержать.

   Правильный код:

      ```java
   // сервис
   public Task save(Task task) {
        Optional<User> userOptional = userService.findById(task.getUserId());
        if (userOptional.isEmpty()) {
            throw new NoSuchElementException("User with this id is not found");
        }
        String title = task.getTitle();
        if (title == null) {
            throw new IllegalArgumentException("Task title must not be empty");
        }
        return taskRepository.save(task);
   }
   // ...
   // контроллер
   public ModelAndView saveTask(@ModelAttribute("task") Task task) {
        try {
            taskService.save(task);
            return new ModelAndView("task/list");
        } catch(NoSuchElementException exception) {
            return new ModelAndView("error/404", "errorMessage", new ExceptionMessage(exception.getMessage()));
        } catch(IllegalArgumentException exception) {
            return new ModelAndView("error/400", "errorMessage", new ExceptionMessage(exception.getMessage()));
        }
   }
   ```

   **Важно**. Если вам уже знакомы темы `@ExceptionHandler` или
   `@ControllerAdvise`, то можете использовать их.

## view

1. Группировка представлений. Группируйте представления по директориям в
   зависимости от доменной модели. Например, для Task
   можно в директории `templates/task` создать представления
   `templates/task/list.html`, `templates/task/creationForm.html` и т.д.;
2. Представления вывода ошибок. Представления, который отвечают за вывод
   ошибок, должны быть в директории `templates/error`;
3. Вывод ошибок. Представление должно содержать текст ошибки и ссылку на
   какую-либо страницу. Например, если пользователь допустил ошибку при
   создании задачи, то можно выводить ссылку, ведущую на форму создания
   задачи, т.к. после полученного сообщения должен уже успешно ее создать. 