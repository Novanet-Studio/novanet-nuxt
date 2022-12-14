<template>
  <container-wrapper>
    <custom-header
      icon="z"
      view="Contacto"
      bg="header--blue"
      title-class="nav__title"
    />
    <main class="main">
      <section class="section">
        <div class="contacto">
          <div class="contacto__items">
            <h2 class="blue">
              {{ contact?.data.attributes.principal.titulo }}
            </h2>
            <div
              v-html="
                markdown.render(
                  contact?.data.attributes.principal.descripcion ?? '',
                )
              "
            ></div>
            <iframe
              src="https://www.google.com/maps/embed?pb=!1m14!1m8!1m3!1d15692.576250424276!2d-66.8779468!3d10.4893076!3m2!1i1024!2i768!4f13.1!3m3!1m2!1s0x0%3A0x1952146d76df33c4!2sNovanet%20Studio!5e0!3m2!1sen!2sve!4v1672924631601!5m2!1sen!2sve"
              class="contacto__mapa"
              allowfullscreen=""
              loading="lazy"
              referrerpolicy="no-referrer-when-downgrade"
            ></iframe>
          </div>

          <div class="contacto__items">
            <h2 class="blue">
              {{ contact?.data.attributes.formulario.titulo }}
            </h2>
            <p>
              {{ contact?.data.attributes.formulario.descripcion }}
            </p>
            <form
              class="formulario"
              name="contacto"
              @submit.prevent="onSubmit"
              action="/gracias/"
              method="post"
              data-netlify="true"
              data-netlify-honeypot="bot-field"
            >
              <h3 class="formulario__titulo">Formulario de contacto</h3>

              <p hidden>
                <label> Don’t fill this out: <input name="bot-field" /> </label>
              </p>
              <div>
                <input
                  type="text"
                  name="name"
                  v-model="name"
                  placeholder="Nombre y Apellido"
                />
              </div>

              <div>
                <input
                  type="email"
                  name="email"
                  v-model="email"
                  placeholder="Email"
                />
              </div>

              <div>
                <input
                  type="number"
                  name="phone"
                  v-model="phone"
                  placeholder="Teléfono"
                />
              </div>

              <div>
                <input
                  type="text"
                  name="origin"
                  v-model="origin"
                  placeholder="¿Cómo nos encontró?"
                />
              </div>

              <div>
                <textarea
                  name="message"
                  v-model="message"
                  placeholder="Mensaje"
                />
              </div>
              <input type="submit" value="Enviar" />
              <input type="hidden" name="form-name" value="inicio" />
            </form>
          </div>
        </div>
      </section>
    </main>
  </container-wrapper>
</template>

<script lang="ts" setup>
import MarkdownIt from 'markdown-it';
import { useForm } from 'vee-validate';
import * as yup from 'yup';

definePageMeta({
  layout: 'page',
});

const clog = (e: any) => {
  console.log(e);
};

const graphql = useStrapiGraphQL();
const contact = ref<Project.Contact>();
const markdown = new MarkdownIt();

useHead({
  title: 'Contacto',
  meta: [
    {
      name: 'description',
      content: contact.value?.data.attributes.principal.descripcion.substring(
        0,
        168,
      ),
    },
  ],
});

const schema = yup.object({
  name: yup.string().required('Este campo es requerido'),
  email: yup.string().email().required('Este campo es requerido'),
  phone: yup.string().required('Este campo es requerido'),
  origin: yup.string().required('Este campo es requerido'),
  message: yup.string().required('Este campo es requerido'),
});

// We can show error message if exist
const {
  useFieldModel,
  errors: _,
  handleSubmit,
} = useForm({
  validationSchema: schema,
});

const [name, email, phone, origin, message] = useFieldModel([
  'name',
  'email',
  'phone',
  'origin',
  'message',
]);

const router = useRouter();

const encode = (data: any) =>
  Object.keys(data)
    .map((key) => encodeURIComponent(key) + '=' + encodeURIComponent(data[key]))
    .join('&');

const onSubmit = handleSubmit(async (values) => {
  try {
    await fetch('/', {
      method: 'POST',
      headers: { 'Content-Type': 'application/x-www-form-urlencoded' },
      body: encode({
        'form-name': 'contacto',
        ...values,
      }),
    });
    router.push('/gracias');
  } catch (error) {
    console.log('An error occurred while sending the form data: ', error);
  }
});

try {
  const response = await graphql<Project.ContactResponse>(`
    query Contacto {
      contacto {
        data {
          attributes {
            principal {
              titulo
              descripcion
            }
            formulario {
              titulo
              descripcion
            }
            latitud
            longitud
          }
        }
      }
    }
  `);

  contact.value = response.data.contacto;
} catch (error) {
  console.error('An error ocurre while fetching contact data: ', error);
}
</script>
