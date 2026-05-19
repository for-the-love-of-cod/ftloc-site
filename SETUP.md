# Setup

One-time configuration required for the Supabase backend.

## Storage: image uploads

The EntryForm uploads selected images to a Supabase Storage bucket named
`entry-images`, then writes the resulting public URL into `entries.image_url`.

### 1. Create the bucket

In the Supabase dashboard:

- **Storage → New bucket**
- Name: `entry-images`
- Public: **on**

### 2. Apply the storage policies

In **SQL editor**, run:

```sql
create policy "Users can upload to their own folder"
  on storage.objects for insert to authenticated
  with check (
    bucket_id = 'entry-images'
    and auth.role() = 'authenticated'
    and (storage.foldername(name))[1] = auth.uid()::text
  );

create policy "Public read of entry images"
  on storage.objects for select to anon, authenticated
  using ( bucket_id = 'entry-images' );

create policy "Users can delete their own images"
  on storage.objects for delete to authenticated
  using (
    bucket_id = 'entry-images'
    and auth.uid()::text = (storage.foldername(name))[1]
  );
```

The upload path the client writes is `<user.id>/<timestamp>-<uuid>.jpg`, so
the first path segment is the uploader's UUID — which is what the INSERT and
DELETE policies pin to. A public SELECT policy is included even though the
bucket is marked public, so the intent is explicit.
